---
title: Memorizzazione del contenuto protetto nella cache
seo-title: Memorizzazione del contenuto protetto nella cache in AEM Dispatcher
description: Scopri come funziona il caching sensibile alle autorizzazioni in Dispatcher.
seo-description: Scoprite come funziona il caching sensibile alle autorizzazioni in AEM Dispatcher.
uuid: abFed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: Utente
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: spedizioniere
content-type: riferimento
discoiquuid: 4f9b2bc8-a309-47bc-b70d-a1c0da78d464
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97

---


# Memorizzazione del contenuto protetto nella cache {#caching-secured-content}

Il caching sensibile alle autorizzazioni consente di memorizzare nella cache le pagine protette. Il dispatcher controlla le autorizzazioni di accesso dell'utente per una pagina prima di distribuirla nella cache.

Dispatcher include il modulo AuthChecker che implementa il caching sensibile alle autorizzazioni. Quando il modulo è attivato, il rendering chiama un servlet AEM per eseguire l’autenticazione utente e l’autorizzazione per il contenuto richiesto. La risposta del servlet determina se il contenuto viene inviato al browser Web.

Poiché i metodi di autenticazione e autorizzazione sono specifici della distribuzione AEM, è necessario creare il servlet.

>[!NOTE]
>
>Utilizzate `deny` i filtri per applicare restrizioni di protezione predefinite. Utilizzate il caching sensibile alle autorizzazioni per le pagine configurate per consentire l'accesso a un sottoinsieme di utenti o gruppi.

I diagrammi seguenti illustrano l'ordine degli eventi che si verificano quando un browser Web richiede una pagina per la quale viene utilizzato il caching sensibile alle autorizzazioni.

## La pagina è memorizzata nella cache e l'utente è autorizzato {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Il dispatcher determina che il contenuto richiesto è memorizzato nella cache e valido.
1. Il dispatcher invia un messaggio di richiesta al rendering. La sezione HEAD include tutte le righe di intestazione della richiesta del browser.
1. Il rendering chiama l’autore per eseguire il controllo di sicurezza e risponde al dispatcher. Il messaggio di risposta include un codice di stato HTTP pari a 200 per indicare che l'utente è autorizzato.
1. Il dispatcher invia un messaggio di risposta al browser composto dalle righe di intestazione della risposta di rendering e dal contenuto memorizzato nella cache del corpo.

## La pagina non è memorizzata nella cache e l'utente è autorizzato {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Il dispatcher determina che il contenuto non è memorizzato nella cache o richiede un aggiornamento.
1. Il dispatcher inoltra la richiesta originale al rendering.
1. Il rendering chiama il servlet dell’autore per eseguire un controllo di sicurezza. Quando l’utente è autorizzato, il rendering include la pagina rappresentata nel corpo del messaggio di risposta.
1. Dispatcher forwards the response to the browser. Dispatcher adds the body of the render's response message to the cache.

## Utente non autorizzato {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Dispatcher checks the cache.
1. Dispatcher sends a request message to the render that includes all header lines from the browser's request.
1. The render calls the authorizer servlet to perform a security check which fails, and the render forwards the original request to Dispatcher.

## Implementing permission-sensitive caching {#implementing-permission-sensitive-caching}

To implement permission-sensitive caching, perform the following tasks:

* Develop a servlet that performs authentication and authorization
* Configure the Dispatcher

>[!NOTE]
>
>Typically, secure resources are stored in a separate folder than unsecure files. For example, /content/secure/


## Create the authorization servlet {#create-the-authorization-servlet}

Create and deploy a servlet that performs the authentication and authorization of the user who requests the web content. The servlet can use any authentication and authorization method, such as the AEM user account and repository ACLs, or an LDAP lookup service. You deploy the servlet to the AEM instance that Dispatcher uses as the render.

Il servlet deve essere accessibile a tutti gli utenti. Pertanto, il servlet deve estendere la `org.apache.sling.api.servlets.SlingSafeMethodsServlet` classe, che fornisce l'accesso in sola lettura al sistema.

Il servlet riceve solo le richieste HEAD dal rendering, quindi è necessario solo implementare il `doHead` metodo.

Il rendering include l’URI della risorsa richiesta come parametro della richiesta HTTP. Ad esempio, un servlet di autorizzazione è accessibile tramite `/bin/permissioncheck`. Per eseguire un controllo di sicurezza nella pagina /content/geometrixx-outdoors/en.html, il rendering include il seguente URL nella richiesta HTTP:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

Il messaggio di risposta del servlet deve contenere i seguenti codici di stato HTTP:

* 200: Autenticazione e autorizzazione passate.

Il servlet di esempio seguente ottiene l’URL della risorsa richiesta dalla richiesta HTTP. Il codice utilizza l' `Property` annotazione Felix SCR per impostare il valore della `sling.servlet.paths` proprietà su /bin/permissioncheck. Nel `doHead` metodo, il servlet ottiene l'oggetto session e utilizza il `checkPermission` metodo per determinare il codice di risposta appropriato.

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

* `url`: Il valore della `sling.servlet.paths` proprietà del servlet che esegue il controllo di sicurezza.

* `filter`: Filtri che specificano le cartelle a cui viene applicato il caching sensibile alle autorizzazioni. In genere, un `deny` filtro viene applicato a tutte le cartelle e `allow` i filtri vengono applicati alle cartelle protette.

* `headers`: Specifica le intestazioni HTTP incluse nel servlet di autorizzazione nella risposta.

All'avvio del dispatcher, il file di registro del dispatcher include il seguente messaggio a livello di debug:

`AuthChecker: initialized with URL 'configured_url'.`

La sezione auth_checker di esempio seguente configura Dispatcher per l’utilizzo del servlet dell’argomento precedente. La sezione del filtro determina l'esecuzione dei controlli delle autorizzazioni solo per le risorse HTML protette.

### Esempio di configurazione {#example-configuration}

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

