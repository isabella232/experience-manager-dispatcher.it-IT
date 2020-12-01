---
title: Caching di contenuto protetto
seo-title: Memorizzazione del contenuto protetto nella cache in AEM Dispatcher
description: Scopri come funziona il caching sensibile alle autorizzazioni in Dispatcher.
seo-description: Scoprite come funziona il caching sensibile alle autorizzazioni in AEM Dispatcher.
uuid: abfed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 4f9b2bc8-a309-47bc-b70d-a1c0da78d464
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97
workflow-type: tm+mt
source-wordcount: '762'
ht-degree: 0%

---


# Caching di contenuto protetto {#caching-secured-content}

Il caching sensibile alle autorizzazioni consente di memorizzare nella cache le pagine protette. Il dispatcher controlla le autorizzazioni di accesso dell&#39;utente per una pagina prima di distribuirla nella cache.

Dispatcher include il modulo AuthChecker che implementa il caching sensibile alle autorizzazioni. Quando il modulo è attivato, il rendering chiama un servlet AEM per eseguire l&#39;autenticazione utente e l&#39;autorizzazione per il contenuto richiesto. La risposta del servlet determina se il contenuto viene inviato al browser Web.

Poiché i metodi di autenticazione e autorizzazione sono specifici per la distribuzione AEM, è necessario creare il servlet.

>[!NOTE]
>
>Utilizzate i filtri `deny` per applicare restrizioni di protezione predefinite. Utilizzate il caching sensibile alle autorizzazioni per le pagine configurate per consentire l&#39;accesso a un sottoinsieme di utenti o gruppi.

I diagrammi seguenti illustrano l&#39;ordine degli eventi che si verificano quando un browser Web richiede una pagina per la quale viene utilizzato il caching sensibile alle autorizzazioni.

## La pagina è memorizzata nella cache e l&#39;utente è autorizzato {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Il dispatcher determina che il contenuto richiesto è memorizzato nella cache e valido.
1. Il dispatcher invia un messaggio di richiesta al rendering. La sezione HEAD include tutte le righe di intestazione della richiesta del browser.
1. Il rendering chiama l’autore per eseguire il controllo di sicurezza e risponde al dispatcher. Il messaggio di risposta include un codice di stato HTTP pari a 200 per indicare che l&#39;utente è autorizzato.
1. Il dispatcher invia un messaggio di risposta al browser composto dalle righe di intestazione della risposta di rendering e dal contenuto memorizzato nella cache del corpo.

## La pagina non è memorizzata nella cache e l&#39;utente è autorizzato {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Il dispatcher determina che il contenuto non è memorizzato nella cache o richiede un aggiornamento.
1. Il dispatcher inoltra la richiesta originale al rendering.
1. Il rendering chiama il servlet dell’autore per eseguire un controllo di sicurezza. Quando l’utente è autorizzato, il rendering include la pagina di cui è stato effettuato il rendering nel corpo del messaggio di risposta.
1. Il dispatcher inoltra la risposta al browser. Dispatcher aggiunge il corpo del messaggio di risposta del rendering alla cache.

## Utente non autorizzato {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Il dispatcher controlla la cache.
1. Il dispatcher invia al rendering un messaggio di richiesta che include tutte le righe di intestazione della richiesta del browser.
1. Il rendering chiama il servlet dell’autore per eseguire un controllo di sicurezza che non riesce e il rendering inoltra la richiesta originale al dispatcher.

## Implementazione del caching sensibile alle autorizzazioni {#implementing-permission-sensitive-caching}

Per implementare il caching sensibile alle autorizzazioni, effettuate le seguenti operazioni:

* Sviluppare un servlet che esegue l&#39;autenticazione e l&#39;autorizzazione
* Configurare il dispatcher

>[!NOTE]
>
>In genere, le risorse protette vengono memorizzate in una cartella separata rispetto ai file non protetti. Ad esempio, /content/secure/


## Creare il servlet di autorizzazione {#create-the-authorization-servlet}

Creare e distribuire un servlet che esegue l&#39;autenticazione e l&#39;autorizzazione dell&#39;utente che richiede il contenuto Web. Il servlet può utilizzare qualsiasi metodo di autenticazione e autorizzazione, ad esempio l&#39;account utente AEM e gli ACL dell&#39;archivio oppure un servizio di ricerca LDAP. Distribuite il servlet nell&#39;istanza AEM utilizzata da Dispatcher come rendering.

Il servlet deve essere accessibile a tutti gli utenti. Pertanto, il servlet deve estendere la classe `org.apache.sling.api.servlets.SlingSafeMethodsServlet`, che fornisce l&#39;accesso in sola lettura al sistema.

Il servlet riceve solo le richieste HEAD dal rendering, pertanto è necessario implementare solo il metodo `doHead`.

Il rendering include l’URI della risorsa richiesta come parametro della richiesta HTTP. Ad esempio, un servlet di autorizzazione è accessibile tramite `/bin/permissioncheck`. Per eseguire un controllo di sicurezza nella pagina /content/geometrixx-outdoors/en.html, il rendering include il seguente URL nella richiesta HTTP:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

Il messaggio di risposta del servlet deve contenere i seguenti codici di stato HTTP:

* 200: Autenticazione e autorizzazione passate.

Il servlet di esempio seguente ottiene l’URL della risorsa richiesta dalla richiesta HTTP. Il codice utilizza l&#39;annotazione Felix SCR `Property` per impostare il valore della proprietà `sling.servlet.paths` su /bin/permissioncheck. Nel metodo `doHead`, il servlet ottiene l&#39;oggetto session e utilizza il metodo `checkPermission` per determinare il codice di risposta appropriato.

>[!NOTE]
>
>Il valore della proprietà sling.servlet.paths deve essere attivato nel servizio Sling Servlet Resolver (org.apache.sling.servlets.resolver.SlingServletResolver).

### Esempio di servlet {#example-servlet}

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.jcr.Session;

@Component(metatype=false)
@Service
public class AuthcheckerServlet extends SlingSafeMethodsServlet {
 
    @Property(value="/bin/permissioncheck")
    static final String SERVLET_PATH="sling.servlet.paths";
    
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    
    public void doHead(SlingHttpServletRequest request, SlingHttpServletResponse response) {
     try{ 
      //retrieve the requested URL
      String uri = request.getParameter("uri");
      //obtain the session from the request
      Session session = request.getResourceResolver().adaptTo(javax.jcr.Session.class);     
      //perform the permissions check
      try {
       session.checkPermission(uri, Session.ACTION_READ);
       logger.info("authchecker says OK");
       response.setStatus(SlingHttpServletResponse.SC_OK);
      } catch(Exception e) {
       logger.info("authchecker says READ access DENIED!");
       response.setStatus(SlingHttpServletResponse.SC_FORBIDDEN);
      }
     }catch(Exception e){
      logger.error("authchecker servlet exception: " + e.getMessage());
     }
    }
}
```

## Configurare il dispatcher per il caching sensibile alle autorizzazioni {#configure-dispatcher-for-permission-sensitive-caching}

La sezione auth_checker del file dispatcher.any controlla il funzionamento del caching sensibile alle autorizzazioni. La sezione auth_checker include le seguenti sottosezioni:

* `url`: Il valore della  `sling.servlet.paths` proprietà del servlet che esegue il controllo di sicurezza.

* `filter`: Filtri che specificano le cartelle a cui viene applicato il caching sensibile alle autorizzazioni. In genere, un filtro `deny` viene applicato a tutte le cartelle e i filtri `allow` vengono applicati alle cartelle protette.

* `headers`: Specifica le intestazioni HTTP incluse nel servlet di autorizzazione nella risposta.

All&#39;avvio del dispatcher, il file di registro del dispatcher include il seguente messaggio a livello di debug:

`AuthChecker: initialized with URL 'configured_url'.`

La sezione auth_checker di esempio seguente configura Dispatcher per l’utilizzo del servlet dell’argomento precedente. La sezione del filtro determina l&#39;esecuzione dei controlli delle autorizzazioni solo per le risorse HTML protette.

### Configurazione esempio {#example-configuration}

```xml
/auth_checker
  {
  # request is sent to this URL with '?uri=<page>' appended
  /url "/bin/permissioncheck"
      
  # only the requested pages matching the filter section below are checked,
  # all other pages get delivered unchecked
  /filter
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "/content/secure/*.html"
      /type "allow"
      }
    }
  # any header line returned from the auth_checker's HEAD request matching
  # the section below will be returned as well
  /headers
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "Set-Cookie:*"
      /type "allow"
      }
    }
  }
```

