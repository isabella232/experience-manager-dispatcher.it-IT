---
title: Annullamento della validità delle pagine memorizzate nella cache da AEM
seo-title: Annullamento della validità delle pagine memorizzate nella cache da Adobe AEM
description: Scoprite come configurare l'interazione tra Dispatcher e AEM per garantire una gestione efficace della cache.
seo-description: Scopri come configurare l'interazione tra Adobe AEM Dispatcher e AEM per garantire una gestione efficace della cache.
uuid: 66533299-55 c 0-4864-9 beb -77 e 281 af 9359
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: Utente
products: SG_ EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: riferimento
discoiquuid: 79 cd 94 be-a 6 bc -4 d 34-bfe 9-393 b 4107925 c
translation-type: tm+mt
source-git-commit: 76cffbfb616cd5601aed36b7076f67a2faf3ed3b

---


# Annullamento della validità delle pagine memorizzate nella cache da AEM {#invalidating-cached-pages-from-aem}

Quando si utilizza Dispatcher con AEM, l&#39;interazione deve essere configurata per garantire una gestione efficace della cache. A seconda dell&#39;ambiente, la configurazione può anche aumentare le prestazioni.

## Configurazione degli account utente AEM {#setting-up-aem-user-accounts}

L&#39;account `admin` utente predefinito viene usato per autenticare gli agenti di replica che vengono installati per impostazione predefinita. Dovreste creare un account utente dedicato da utilizzare con gli agenti di replica. [](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps)

Per ulteriori informazioni, consulta la [sezione Configura utenti](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps) replica e trasporto dell&#39;elenco di controllo AEM Security.

## Annullamento della validità cache del dispatcher dall&#39;ambiente Authoring {#invalidating-dispatcher-cache-from-the-authoring-environment}

Un agente di replica nell&#39;istanza di creazione AEM invia una richiesta di annullamento della validità alla cache al momento della pubblicazione di una pagina. La richiesta fa sì che il dispatcher aggiorni il file nella cache durante la pubblicazione del nuovo contenuto.

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

Utilizzate la seguente procedura per configurare un agente di replica sull&#39;istanza di autore AEM per annullare la cache del dispatcher all&#39;attivazione della pagina:

1. Aprite la console Strumenti AEM. (`https://localhost:4502/miscadmin#/etc`)
1. Apri l&#39;agente di replica richiesto in Strumenti/replica/Agenti nell&#39;autore. Potete utilizzare l&#39;agente Svuotatore del dispatcher che viene installato per impostazione predefinita.
1. Fate clic su Modifica, quindi, nella scheda Impostazioni, verificate **che Attivato** sia selezionato.

1. (Facoltativo) Per abilitare le richieste di annullamento del percorso alias o personalizzato, selezionate l&#39; **opzione di aggiornamento** Alias.
1. Nella scheda Trasporto, immettete l&#39;URI necessario per accedere al dispatcher.\
   Se utilizzi l&#39;agente standard Svuotatore di Dispatcher, probabilmente dovrai aggiornare il nome host e la porta; ad esempio, https:// &lt;*dispatcherhost*&gt;: &lt;*Portapache*&gt;/dispatcher/invalidate. cache

   **Nota:** Per gli agenti Svuotatore del dispatcher, la proprietà URI viene utilizzata solo se utilizzate voci virtualhost basate sul percorso per distinguere tra le aziende. Utilizzate questo campo per eseguire il targeting della fattoria per invalidare. Ad esempio, la versione farm # 1 dispone di un host virtuale e `www.mysite.com/path1/*` di una fattoria # `www.mysite.com/path2/*`2. Potete usare un URL di `/path1/invalidate.cache` destinazione per la prima fattoria e `/path2/invalidate.cache` impostare come destinazione la seconda fattoria. For more information, see [Using Dispatcher with Multiple Domains](dispatcher-domains.md).

1. Configurate gli altri parametri come necessario.
1. Fate clic su OK per attivare l&#39;agente.

In alternativa, puoi anche accedere e configurare l&#39;agente di svuotamento del dispatcher dall&#39;interfaccia utente di [AEM Touch](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent).

Per ulteriori dettagli su come abilitare l&#39;accesso agli URL personalizzati, consultate [Abilitazione dell&#39;accesso agli URL personalizzati](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls).

>[!NOTE]
>
>L&#39;agente per la cancellazione della cache del dispatcher non deve avere un nome utente e una password, ma se configurato verrà inviato con autenticazione di base.

Esistono due potenziali problemi con questo approccio:

* Il dispatcher deve essere raggiungibile dall&#39;istanza di authoring. Se la rete (ad es. il firewall) è configurata in modo che l&#39;accesso tra i due è limitato, ciò potrebbe non esserlo.

* La pubblicazione e la validità della cache avvengono contemporaneamente. A seconda del tempistico, un utente può richiedere una pagina subito dopo essere stata rimossa dalla cache e subito prima della pubblicazione della nuova pagina. AEM ora restituisce la vecchia pagina e il Dispatcher la memorizza nuovamente nella cache. Si tratta di un problema più grave per i siti di grandi dimensioni.

## Annullamento della validità cache del dispatcher da un&#39;istanza di pubblicazione {#invalidating-dispatcher-cache-from-a-publishing-instance}

In determinate circostanze, è possibile ottenere i migliori miglioramenti in termini di prestazioni trasferendo la gestione della cache dall&#39;ambiente di authoring a un&#39;istanza di pubblicazione. Sarà quindi l&#39;ambiente di pubblicazione (non l&#39;ambiente di authoring AEM) che invia una richiesta di annullamento della validità a Dispatcher quando viene ricevuta una pagina pubblicata.

Queste situazioni includono:

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* Impedire possibili conflitti temporali tra Dispatcher e l&#39;istanza di pubblicazione (consultate [Annullamento della cache del dispatcher dall&#39;ambiente Authoring](#invalidating-dispatcher-cache-from-the-authoring-environment)).
* Il sistema include diverse istanze di pubblicazione che risiedono su server ad alte prestazioni e una sola istanza di authoring.

>[!NOTE]
>
>La decisione di utilizzare questo metodo deve essere eseguita da un amministratore AEM esperto.

Il dispatcher viene controllato da un agente di replica che opera nell&#39;istanza di pubblicazione. Tuttavia, la configurazione viene eseguita nell&#39;ambiente di authoring e quindi trasferite mediante l&#39;attivazione dell&#39;agente:

1. Aprite la console Strumenti AEM.
1. Apri l&#39;agente di replica necessario in Strumenti/replica/Agenti durante la pubblicazione. Potete utilizzare l&#39;agente Svuotatore del dispatcher che viene installato per impostazione predefinita.
1. Fate clic su Modifica, quindi, nella scheda Impostazioni, verificate **che Attivato** sia selezionato.
1. (Facoltativo) Per abilitare le richieste di annullamento del percorso alias o personalizzato, selezionate l&#39; **opzione di aggiornamento** Alias.
1. Nella scheda Trasporto, immettete l&#39;URI necessario per accedere al dispatcher.\
   Se utilizzi l&#39;agente standard Svuotatore di Dispatcher, probabilmente dovrai aggiornare il nome host e la porta; ad esempio, `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **Nota:** Per gli agenti Svuotatore del dispatcher, la proprietà URI viene utilizzata solo se utilizzate voci virtualhost basate sul percorso per distinguere tra le aziende. Utilizzate questo campo per eseguire il targeting della fattoria per invalidare. Ad esempio, la versione farm # 1 dispone di un host virtuale e `www.mysite.com/path1/*` di una fattoria # `www.mysite.com/path2/*`2. Potete usare un URL di `/path1/invalidate.cache` destinazione per la prima fattoria e `/path2/invalidate.cache` impostare come destinazione la seconda fattoria. For more information, see [Using Dispatcher with Multiple Domains](dispatcher-domains.md).

1. Configurate gli altri parametri come necessario.
1. Ripetere la procedura per ogni istanza di pubblicazione interessata.

Dopo la configurazione, quando si attiva una pagina dall&#39;autore alla pubblicazione, questo agente avvia una replica standard. Il registro include messaggi che indicano richieste provenienti dal server di pubblicazione, simile al seguente esempio:

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## Annullamento manuale della cache del dispatcher {#manually-invalidating-the-dispatcher-cache}

Per annullare (o svuotare) la cache del dispatcher senza attivare una pagina, è possibile emettere una richiesta HTTP al dispatcher. Ad esempio, potete creare un&#39;applicazione AEM che consente agli amministratori o ad altre applicazioni di svuotare la cache.

La richiesta HTTP causa l&#39;eliminazione di file specifici dalla cache in Dispatcher. Facoltativamente, il dispatcher aggiorna la cache con una nuova copia.

### Eliminare i file memorizzati nella cache {#delete-cached-files}

Segnalare una richiesta HTTP che causa l&#39;eliminazione di file dalla cache in Dispatcher. Il dispatcher memorizza nuovamente i file solo quando riceve una richiesta client per la pagina. L&#39;eliminazione dei file memorizzati nella cache è appropriata per i siti Web che non riceveranno richieste simultanee per la stessa pagina.

La richiesta HTTP ha il seguente modulo:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Il dispatcher elimina (elimina) i file memorizzati nella cache e le cartelle con nomi corrispondenti al valore dell&#39; `CQ-Handler` intestazione. Ad esempio, un `CQ-Handle` valore `/content/geomtrixx-outdoors/en` corrisponde ai seguenti elementi:

* Tutti i file (di qualsiasi estensione) denominati `en` nella `geometrixx-outdoors` directory

* Qualsiasi directory denominata &quot; `_jcr_content`&quot; sotto la directory en (che, se esiste, contiene rendering memorizzati nella cache dei nodi della pagina)

Tutti gli altri file nella cache di dispatcher (o fino a un livello particolare, a seconda `/statfileslevel` dell&#39;impostazione) sono invalidati toccando il `.stat` file. La data dell&#39;ultima modifica del file viene confrontata con l&#39;ultima data di modifica di un documento memorizzato nella cache e il documento viene recapitato se il `.stat` file è più recente. Per ulteriori informazioni, vedere [Annullamento della validità dei file per livello](dispatcher-configuration.md#main-pars_title_26) cartella.

L&#39;inconvalida (ovvero il tocco di file. stat) può essere impedita inviando un&#39;intestazione `CQ-Action-Scope: ResourceOnly`aggiuntiva. Questo può essere utilizzato per svuotare risorse particolari senza invalidare altre parti della cache, come i dati JSON che vengono creati in modo dinamico e richiedono l&#39;eliminazione regolare indipendente dalla cache (ad es. se rappresentano dati ottenuti da un sistema di terze parti per visualizzare notizie, ticker di stock ecc.).

### Eliminare e reinserire i file {#delete-and-recache-files}

Inviate una richiesta HTTP che causa l&#39;eliminazione di file memorizzati nella cache da parte del dispatcher e recupera e riapra immediatamente il file. Eliminate e riprendete immediatamente i file quando è probabile che i siti Web ricevano richieste client simultanee per la stessa pagina. La riclassificazione immediata assicura che il dispatcher recuperi e memorizzi la pagina solo una volta, anziché una sola per ogni richiesta client simultanea.

**Nota:** L&#39;eliminazione e la ricommercializzazione dei file devono essere eseguite solo nell&#39;istanza di pubblicazione. Quando viene eseguito dall&#39;istanza di authoring, si verificano condizioni race quando si verificano tentativi di ridisegnare le risorse prima che siano state pubblicate.

La richiesta HTTP ha il seguente modulo:

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

I percorsi di pagina da eliminare immediatamente sono elencati su righe separate nel corpo del messaggio. Il valore di `CQ-Handle` è il percorso di una pagina che invalida le pagine da eliminare. (Vedete il `/statfileslevel` parametro [dell&#39;elemento](dispatcher-configuration.md#main-pars_146_44_0010) di configurazione Cache.) L&#39;esempio di richiesta HTTP seguente elimina e rimuove i `/content/geometrixx-outdoors/en.html page`seguenti elementi:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### Esempio di servlet sfsh {#example-flush-servlet}

Il codice seguente implementa un servlet che invia una richiesta invalidata al dispatcher. Il servlet riceve un messaggio di richiesta che contiene `handle` e `page` parametri. Tali parametri forniscono rispettivamente il valore dell&#39; `CQ-Handle` intestazione e il percorso della pagina. Il servlet utilizza i valori per costruire la richiesta HTTP per Dispatcher.

Quando il servlet viene distribuito nell&#39;istanza di pubblicazione, l&#39;URL seguente causa la cancellazione della pagina /content/geometrixx-outdoors/en.html da parte del dispatcher e quindi la cache di una nuova copia.

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>Questo servlet di esempio non è protetto e mostra solo l&#39;utilizzo del messaggio di richiesta Post HTTP. La soluzione deve proteggere l&#39;accesso al servlet.


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

