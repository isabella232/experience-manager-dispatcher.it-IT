---
title: Annullamento della validità delle pagine in cache da AEM
seo-title: Annullamento Della Convalida Delle Pagine Nella Cache Da  Adobe AEM
description: Scopri come configurare l’interazione tra Dispatcher e AEM per garantire un’efficace gestione della cache.
seo-description: Scoprite come configurare l'interazione tra  Adobe AEM Dispatcher e AEM per garantire un'efficace gestione della cache.
uuid: 66533299-55c0-4864-9beb-77e281af9359
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 79cd94be-a6bc-4d34-bfe9-393b4107925c
translation-type: tm+mt
source-git-commit: 85497651ce29c8564da4b52c60819a48b776af7b
workflow-type: tm+mt
source-wordcount: '1427'
ht-degree: 0%

---


# Annullamento della validità delle pagine in cache da AEM {#invalidating-cached-pages-from-aem}

Quando si utilizza Dispatcher con AEM, l&#39;interazione deve essere configurata per garantire un&#39;efficace gestione della cache. A seconda dell&#39;ambiente, la configurazione può anche migliorare le prestazioni.

## Configurazione AEM account utente {#setting-up-aem-user-accounts}

L&#39;account utente predefinito `admin` viene utilizzato per autenticare gli agenti di replica installati per impostazione predefinita. È necessario creare un account utente dedicato da utilizzare con gli agenti di replica.

Per ulteriori informazioni, vedere la sezione [Configura utenti replica e trasporto](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps) dell&#39;elenco di controllo AEM sicurezza.

## Annullamento della validità della cache del dispatcher dall&#39;ambiente di authoring {#invalidating-dispatcher-cache-from-the-authoring-environment}

Un agente di replica nell’istanza di creazione AEM invia una richiesta di annullamento validità cache al Dispatcher quando viene pubblicata una pagina. La richiesta causa l&#39;aggiornamento del file nella cache da parte del Dispatcher durante la pubblicazione del nuovo contenuto.

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-0436B4A35714BFF67F000101@AdobeID)
Last Modified Date: 2017-05-25T10:37:23.679-0400

<p>Hiding this information due to <a href="https://jira.corp.adobe.com/browse/CQDOC-5805">CQDOC-5805</a>.</p>

 -->

Per configurare un agente di replica nell’istanza di creazione AEM in modo da annullare la validità della cache del dispatcher al momento dell’attivazione della pagina, procedere come segue:

1. Aprite la console Strumenti AEM. (`https://localhost:4502/miscadmin#/etc`)
1. Aprire l&#39;agente di replica richiesto in Strumenti/replica/Agenti sull&#39;autore. È possibile utilizzare l&#39;agente Dispatcher Flush installato per impostazione predefinita.
1. Fare clic su Modifica e, nella scheda Impostazioni, assicurarsi che sia selezionato **Enabled**.

1. (Facoltativo) Per abilitare le richieste di annullamento di alias o percorsi personalizzati, selezionare l&#39;opzione **Aggiornamento alias**.
1. Nella scheda Trasporto, immettete l’URI necessario per accedere al Dispatcher.\
   Se utilizzi l’agente standard Dispatcher Flush, probabilmente dovrai aggiornare il nome host e la porta; ad esempio, https://&lt;*dispatcherHost*:&lt;*portApache*>/dispatcher/invalidate.cache

   **Nota:** per gli agenti Dispatcher Flush, la proprietà URI viene utilizzata solo se si utilizzano voci virtualhost basate sul percorso per distinguere tra farm. Utilizzate questo campo per eseguire il targeting della farm per annullare la validità. Ad esempio, la farm n. 1 ha un host virtuale di `www.mysite.com/path1/*` e la farm n. 2 ha un host virtuale di `www.mysite.com/path2/*`. Potete utilizzare un URL di `/path1/invalidate.cache` per impostare come destinazione la prima farm e `/path2/invalidate.cache` per la seconda farm. Per ulteriori informazioni, vedere [Utilizzo del dispatcher con più domini](dispatcher-domains.md).

1. Configurate gli altri parametri come necessario.
1. Fate clic su OK per attivare l’agente.

In alternativa, puoi anche accedere e configurare l&#39;agente Dispatcher Flush dall&#39; [AEM Touch UI](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent).

Per ulteriori dettagli su come abilitare l&#39;accesso agli URL personalizzati, consultate [Abilitazione dell&#39;accesso agli URL personalizzati](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls).

>[!NOTE]
>
>L&#39;agente per lo scaricamento della cache del dispatcher non deve avere un nome utente e una password, ma se configurato sarà inviato con l&#39;autenticazione di base.

Questo approccio presenta due possibili problemi:

* Il dispatcher deve essere raggiungibile dall’istanza di creazione. Se la rete (ad es. il firewall) è configurato in modo tale da limitare l’accesso tra i due, l’errore potrebbe non essere questo.

* La pubblicazione e l&#39;annullamento della validità della cache avvengono contemporaneamente. A seconda del tempo, un utente può richiedere una pagina subito dopo che è stata rimossa dalla cache e subito prima della pubblicazione della nuova pagina. AEM ora restituisce la vecchia pagina e il Dispatcher la memorizza nuovamente nella cache. Si tratta più di un problema per i siti di grandi dimensioni.

## Annullamento della validità della cache del dispatcher da un&#39;istanza di pubblicazione {#invalidating-dispatcher-cache-from-a-publishing-instance}

In alcune circostanze, è possibile ottenere dei vantaggi in termini di prestazioni trasferendo la gestione della cache dall’ambiente di authoring a un’istanza di pubblicazione. Sarà quindi l’ambiente di pubblicazione (non l’ambiente di authoring AEM) a inviare una richiesta di annullamento validità cache al Dispatcher quando viene ricevuta una pagina pubblicata.

Tali circostanze comprendono:

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* Prevenzione di possibili conflitti temporali tra il dispatcher e l&#39;istanza di pubblicazione (vedere [Invalidazione della cache del dispatcher dall&#39;ambiente di authoring](#invalidating-dispatcher-cache-from-the-authoring-environment)).
* Il sistema include diverse istanze di pubblicazione che risiedono su server ad alte prestazioni e una sola istanza di authoring.

>[!NOTE]
>
>La decisione di utilizzare questo metodo dovrebbe essere presa da un amministratore AEM esperto.

Lo scaricamento del dispatcher è controllato da un agente di replica che opera nell’istanza di pubblicazione. Tuttavia, la configurazione viene eseguita nell’ambiente di authoring e quindi trasferita attivando l’agente:

1. Aprite la console Strumenti AEM.
1. Aprite l&#39;agente di replica richiesto sotto Strumenti/replica/Agenti al momento della pubblicazione. È possibile utilizzare l&#39;agente Dispatcher Flush installato per impostazione predefinita.
1. Fare clic su Modifica e, nella scheda Impostazioni, assicurarsi che sia selezionato **Enabled**.
1. (Facoltativo) Per abilitare le richieste di annullamento di alias o percorsi personalizzati, selezionare l&#39;opzione **Aggiornamento alias**.
1. Nella scheda Trasporto, immettete l’URI necessario per accedere al Dispatcher.\
   Se utilizzi l’agente standard Dispatcher Flush, probabilmente dovrai aggiornare il nome host e la porta; ad esempio, `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **Nota:** per gli agenti Dispatcher Flush, la proprietà URI viene utilizzata solo se si utilizzano voci virtualhost basate sul percorso per distinguere tra farm. Utilizzate questo campo per eseguire il targeting della farm per annullare la validità. Ad esempio, la farm n. 1 ha un host virtuale di `www.mysite.com/path1/*` e la farm n. 2 ha un host virtuale di `www.mysite.com/path2/*`. Potete utilizzare un URL di `/path1/invalidate.cache` per impostare come destinazione la prima farm e `/path2/invalidate.cache` per la seconda farm. Per ulteriori informazioni, vedere [Utilizzo del dispatcher con più domini](dispatcher-domains.md).

1. Configurate gli altri parametri come necessario.
1. Ripetete questa operazione per ogni istanza di pubblicazione interessata.

Dopo la configurazione, quando si attiva una pagina dall’autore alla pubblicazione, questo agente avvia una replica standard. Il registro contiene messaggi che indicano le richieste provenienti dal server di pubblicazione, in modo simile al seguente esempio:

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## Annullamento manuale della validità della cache del dispatcher {#manually-invalidating-the-dispatcher-cache}

Per annullare (o cancellare) la cache del dispatcher senza attivare una pagina, è possibile inviare una richiesta HTTP al dispatcher. Ad esempio, potete creare un&#39;applicazione AEM che consenta agli amministratori o ad altre applicazioni di cancellare la cache.

La richiesta HTTP causa l’eliminazione di file specifici dalla cache da parte del dispatcher. Facoltativamente, il dispatcher aggiorna la cache con una nuova copia.

### Eliminare i file memorizzati nella cache {#delete-cached-files}

Eseguite una richiesta HTTP che causa l&#39;eliminazione dei file dalla cache da parte del dispatcher. Il dispatcher memorizza nuovamente i file nella cache solo quando riceve una richiesta client per la pagina. L’eliminazione dei file memorizzati nella cache in questo modo è appropriata per i siti Web che probabilmente non riceveranno richieste simultanee per la stessa pagina.

La richiesta HTTP presenta il seguente modulo:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Il dispatcher elimina (elimina) i file e le cartelle memorizzati nella cache con nomi che corrispondono al valore dell&#39;intestazione `CQ-Handler`. Ad esempio, un `CQ-Handle` di `/content/geomtrixx-outdoors/en` corrisponde ai seguenti elementi:

* Tutti i file (di qualsiasi estensione di file) denominati `en` nella directory `geometrixx-outdoors`

* Qualsiasi directory denominata &quot; `_jcr_content`&quot; sotto la directory en (che, se esiste, contiene rendering memorizzati nella cache dei sottonodi della pagina)

Tutti gli altri file presenti nella cache del dispatcher (o fino a un determinato livello, a seconda dell&#39;impostazione `/statfileslevel`) vengono invalidati toccando il file `.stat`. L&#39;ultima data di modifica del file viene confrontata con l&#39;ultima data di modifica di un documento memorizzato nella cache e il documento viene recuperato se il file `.stat` è più recente. Per ulteriori informazioni, vedere [Invalidazione dei file per livello di cartella](dispatcher-configuration.md#main-pars_title_26).

L&#39;annullamento della validità (ad es. il tocco di file .stat) può essere impedito inviando un&#39;altra intestazione `CQ-Action-Scope: ResourceOnly`. Questo può essere utilizzato per cancellare risorse particolari senza invalidare altre parti della cache, come i dati JSON creati in modo dinamico e che richiedono lo scaricamento regolare indipendentemente dalla cache (ad esempio, la rappresentazione dei dati ottenuti da un sistema di terze parti per visualizzare notizie, ticker di magazzino, ecc.).

### Eliminare e ricache i file {#delete-and-recache-files}

Emettere una richiesta HTTP che causa l’eliminazione dei file memorizzati nella cache da parte del dispatcher, nonché il recupero e il recupero immediato del file. Eliminate e memorizzate immediatamente nella cache i file quando è probabile che i siti Web ricevano richieste client simultanee per la stessa pagina. La funzionalità di ricognizione immediata assicura che il dispatcher recuperi e memorizzi nella cache la pagina una sola volta, anziché una sola volta per ciascuna delle richieste client simultanee.

**Nota:** l’eliminazione e la memorizzazione dei file nella cache devono essere eseguite solo sull’istanza di pubblicazione. Quando eseguite dall’istanza di creazione, si verificano condizioni di gara quando si tenta di recuperare le risorse prima che siano state pubblicate.

La richiesta HTTP presenta il seguente modulo:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate   
`Content-Type: text/plain  
CQ-Handle: path-pattern  
Content-Length: numchars in bodypage_path0

page_path1 
...  
page_pathn
```

I percorsi di pagina per tornare immediatamente a un&#39;area inferiore sono elencati su righe separate nel corpo del messaggio. Il valore di `CQ-Handle` è il percorso di una pagina che invalida le pagine da recuperare. (Vedere il parametro `/statfileslevel` dell&#39;elemento di configurazione [Cache](dispatcher-configuration.md#main-pars_146_44_0010).) L&#39;esempio seguente di messaggio di richiesta HTTP elimina e richiama il messaggio `/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### Esempio di servlet di scarico {#example-flush-servlet}

Il codice seguente implementa un servlet che invia una richiesta di annullamento della validità al Dispatcher. Il servlet riceve un messaggio di richiesta contenente i parametri `handle` e `page`. Questi parametri forniscono il valore dell&#39;intestazione `CQ-Handle` e il percorso rispettivamente della pagina da spostare. Il servlet utilizza i valori per creare la richiesta HTTP per il Dispatcher.

Quando il servlet viene distribuito nell’istanza di pubblicazione, il seguente URL causa l’eliminazione della pagina /content/geometrixx-outdoors/en.html e quindi la memorizzazione nella cache di una nuova copia.

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>Questo servlet di esempio non è sicuro e dimostra solo l&#39;utilizzo del messaggio di richiesta HTTP Post. La soluzione deve garantire l&#39;accesso al servlet.


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

import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.methods.StringRequestEntity;

@Component(metatype=true)
@Service
public class Flushcache extends SlingSafeMethodsServlet {

 @Property(value="/bin/flushcache")
 static final String SERVLET_PATH="sling.servlet.paths";

 private Logger logger = LoggerFactory.getLogger(this.getClass());

 public void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) {
  try{ 
      //retrieve the request parameters
      String handle = request.getParameter("handle");
      String page = request.getParameter("page");

      //hard-coding connection properties is a bad practice, but is done here to simplify the example
      String server = "localhost"; 
      String uri = "/dispatcher/invalidate.cache";

      HttpClient client = new HttpClient();

      PostMethod post = new PostMethod("https://"+host+uri);
      post.setRequestHeader("CQ-Action", "Activate");
      post.setRequestHeader("CQ-Handle",handle);
   
      StringRequestEntity body = new StringRequestEntity(page,null,null);
      post.setRequestEntity(body);
      post.setRequestHeader("Content-length", String.valueOf(body.getContentLength()));
      client.executeMethod(post);
      post.releaseConnection();
      //log the results
      logger.info("result: " + post.getResponseBodyAsString());
      }
  }catch(Exception e){
      logger.error("Flushcache servlet exception: " + e.getMessage());
  }
 }
}
```

