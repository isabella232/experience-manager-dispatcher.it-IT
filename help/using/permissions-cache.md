---
title: Memorizzazione nella cache del contenuto protetto
seo-title: Memorizzazione nella cache del contenuto protetto in AEM Dispatcher
description: Scoprite come il caching sensibile alle autorizzazioni funziona in Dispatcher.
seo-description: Scoprite come il caching sensibile alle autorizzazioni funziona in AEM Dispatcher.
uuid: abfed 68 a -2 efe -45 f 6-bdf 7-2284931629 d 6
contentOwner: Utente
products: SG_ EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: riferimento
discoiquuid: 4 f 9 b 2 bc 8-a 309-47 bc-b 70 d-a 1 c 0 da 78 d 464
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97

---


# Memorizzazione nella cache del contenuto protetto {#caching-secured-content}

La memorizzazione nella cache sensibile alle autorizzazioni consente di memorizzare nella cache le pagine protette. Il dispatcher controlla le autorizzazioni di accesso dell&#39;utente per una pagina prima di distribuire la pagina memorizzata nella cache.

Il dispatcher include il modulo authchecker che implementa il caching sensibile alle autorizzazioni. Quando il modulo viene attivato, il rendering chiama un servlet AEM per eseguire l&#39;autenticazione e l&#39;autorizzazione utente per il contenuto richiesto. La risposta servlet determina se il contenuto viene distribuito al browser Web.

Poiché i metodi di autenticazione e autorizzazione sono specifici della distribuzione AEM, dovete creare il servlet.

>[!NOTE]
>
>Utilizzate `deny` i filtri per applicare restrizioni di sicurezza. Utilizzare caching sensibile alle autorizzazioni per le pagine configurate per consentire l&#39;accesso a un sottoinsieme di utenti o gruppi.

I diagrammi seguenti illustrano l&#39;ordine degli eventi che si verificano quando un browser Web richiede una pagina per la quale viene utilizzata la cache sensibile alle autorizzazioni.

## La pagina è memorizzata nella cache e l&#39;utente è autorizzato {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Il dispatcher determina che il contenuto richiesto è memorizzato nella cache e valido.
1. Dispatcher invia un messaggio di richiesta al rendering. La sezione HEAD include tutte le righe di intestazione dalla richiesta del browser.
1. Il rendering chiama l&#39;authorizer per eseguire il controllo della sicurezza e risponde al dispatcher. Il messaggio di risposta include un codice di stato HTTP 200 per indicare che l&#39;utente è autorizzato.
1. Dispatcher invia un messaggio di risposta al browser composto dalle righe di intestazione dalla risposta di rendering e dal contenuto memorizzato nella cache del corpo.

## La pagina non è memorizzata nella cache e l&#39;utente è autorizzato {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Il dispatcher determina che il contenuto non è memorizzato nella cache o che richiede l&#39;aggiornamento.
1. Il dispatcher inoltra la richiesta originale al rendering.
1. Il rendering chiama il servlet dell&#39;authoring per eseguire un controllo di sicurezza. Quando l&#39;utente è autorizzato, il rendering include la pagina rappresentata nel corpo del messaggio di risposta.
1. Dispatcher inoltra la risposta al browser. Dispatcher aggiunge il corpo del messaggio di risposta del rendering alla cache.

## L&#39;utente non è autorizzato {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Il dispatcher controlla la cache.
1. Dispatcher invia un messaggio di richiesta al rendering che include tutte le righe di intestazione dalla richiesta del browser.
1. Il rendering chiama il servlet dell&#39;authoring per eseguire una verifica di sicurezza che non riesce e il rendering inoltra la richiesta originale al dispatcher.

## Implementazione di caching sensibile alle autorizzazioni {#implementing-permission-sensitive-caching}

Per implementare caching sensibile alle autorizzazioni, eseguire le operazioni seguenti:

* Sviluppare un servlet che esegua l&#39;autenticazione e l&#39;autorizzazione
* Configurare il dispatcher

>[!NOTE]
>
>In genere, le risorse sicure vengono memorizzate in una cartella separata rispetto a quelle non sicure. Ad esempio, /content/secure/


## Creare il servlet di autorizzazione {#create-the-authorization-servlet}

Create e distribuite un servlet che esegue l&#39;autenticazione e l&#39;autorizzazione dell&#39;utente che richiede il contenuto Web. Il servlet può utilizzare qualsiasi metodo di autenticazione e autorizzazione, ad esempio l&#39;account utente AEM e gli ACL archivio o un servizio di ricerca LDAP. Distribuite il servlet nell&#39;istanza AEM che il dispatcher usa come rendering.

Il servlet deve essere accessibile a tutti gli utenti. Pertanto, il servlet deve estendere la `org.apache.sling.api.servlets.SlingSafeMethodsServlet` classe, che fornisce accesso in sola lettura al sistema.

Il servlet ricalcola solo le richieste HEAD dal rendering, pertanto è necessario implementare il `doHead` metodo.

Il rendering include l&#39;URI della risorsa richiesta come parametro della richiesta HTTP. Ad esempio, viene eseguito l&#39;accesso a un servlet di autorizzazione `/bin/permissioncheck`. Per eseguire un controllo di sicurezza nella pagina /content/geometrixx-outdoors/en.html, il rendering include il seguente URL nella richiesta HTTP:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

Il messaggio di risposta servlet deve contenere i seguenti codici di stato HTTP:

* 200: Autenticazione e autorizzazione inoltrata.

Il seguente servlet di esempio ottiene l&#39;URL della risorsa richiesta dalla richiesta HTTP. Il codice utilizza l&#39;annotazione Felix SCR `Property` per impostare il valore della `sling.servlet.paths` proprietà su /bin/permissioncheck. Nel `doHead` metodo, il servlet ottiene l&#39;oggetto session e utilizza il `checkPermission` metodo per determinare il codice di risposta appropriato.

>[!NOTE]
>
>Il valore della proprietà sling. servlet. paths deve essere attivato nel servizio Resollet Servlet Resolver (org. apache. sling. servlets. resolver. resolver).

### Servlet di esempio {#example-servlet}

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

## Configurare il dispatcher per caching sensibile alle autorizzazioni {#configure-dispatcher-for-permission-sensitive-caching}

La sezione auth_ checker del dispatcher controlla il comportamento del caching sensibile alle autorizzazioni. La sezione auth_ checker include le seguenti sottosezioni:

* `url`: Il valore della `sling.servlet.paths` proprietà del servlet che esegue il controllo di sicurezza.

* `filter`: Filtri che specificano le cartelle a cui viene applicata la cache sensibile alle autorizzazioni. In genere, un `deny` filtro viene applicato a tutte le cartelle e `allow` i filtri vengono applicati alle cartelle protette.

* `headers`: Specifica le intestazioni HTTP che il servlet di autorizzazione include nella risposta.

Quando viene avviato il dispatcher, il file di registro del dispatcher include il seguente messaggio di debug:

`AuthChecker: initialized with URL 'configured_url'.`

La seguente sezione di esempio auth_ checker configura dispatcher per utilizzare il servlet dell&#39;argomento prevoius. La sezione filtro causa la verifica delle autorizzazioni solo per le risorse HTML protette.

### Configurazione di esempio {#example-configuration}

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

