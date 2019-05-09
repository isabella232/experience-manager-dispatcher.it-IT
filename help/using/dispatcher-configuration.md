---
title: Configurazione del dispatcher
seo-title: Configurazione del dispatcher
description: Scopri come configurare Dispatcher.
seo-description: Scopri come configurare Dispatcher.
uuid: 253 ef 0 f 7-2491-4 cec-ab 22-97439 df 29 fd 6
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: '1193211344162'
topic-tags: dispatcher
content-type: riferimento
discoiquuid: aeffee 8 e-bb 34-42 a 7-9 a 5 e-b 7 d 0 e 848391 a
translation-type: tm+mt
source-git-commit: bd8fff69a9c8a32eade60c68fc75c3aa411582af

---


# Configurazione del dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporata nella documentazione per una versione precedente di AEM.

Nelle sezioni seguenti viene descritto come configurare vari aspetti del dispatcher.

## Supporto per ipv 4 e ipv 6 {#support-for-ipv-and-ipv}

Tutti gli elementi di AEM e Dispatcher possono essere installati sia nelle reti ipv 4 che ipv 6. Consultate [IPV 4 e IPV 6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes).

## File di configurazione del dispatcher {#dispatcher-configuration-files}

Per impostazione predefinita, la configurazione di Dispatcher viene memorizzata nel file `dispatcher.any` di testo, anche se potete modificare il nome e la posizione del file durante l&#39;installazione.

Il file di configurazione contiene una serie di proprietà con valori singolo o con più valutazioni che controllano il funzionamento del dispatcher:

* I nomi delle proprietà hanno il prefisso barra `/`.
* Le proprietà con più valori racchiudono gli elementi secondari utilizzando le parentesi `{ }`graffe.

Una configurazione di esempio è strutturata come segue:

```xml
# name of the dispatcher
/name "internet-server"

# each farm configures a set off (loadbalanced) renders
/farms
 {
  # first farm entry (label is not important, just for your convenience)
   /website 
     {  
     /clientheaders
       {
       # List of headers that are passed on
       }
     /virtualhosts
       {
       # List of URLs for this Web site
       }
     /sessionmanagement 
       {
       # settings for user authentification
       }
     /renders
       {
       # List of AEM instances that render the documents
       }
     /filter
       {
       # List of filters
       }
     /vanity_urls
       {
       # List of vanity URLs
       }
     /cache
       {
       # Cache configuration
       /rules
         {
         # List of cachable documents
         }
       /invalidate
         {
         # List of auto-invalidated documents
         }
       }
     /statistics
       {
       /categories
         {
         # The document categories that are used for load balancing estimates
         }
       }
     /stickyConnectionsFor "/myFolder"
     /health_check
       {
       # Page gets contacted when an instance returns a 500
       }
     /retryDelay "1"
     /numberOfRetries "5"
     /unavailablePenalty "1"
     /failover "1"
     }
 }
```

Potete includere altri file che contribuiscono alla configurazione:

* Se il file di configurazione è di grandi dimensioni, potete dividerlo in più file di dimensioni più ridotte (facili da gestire), quindi includerli.
* Per includere file generati automaticamente.

Ad esempio, per includere il file myfarm. Qualsiasi nella configurazione /farms utilizza il seguente codice:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Utilizzate l&#39;asterisco (&quot; *&quot;) come carattere jolly per specificare una serie di file da includere.

Ad esempio, se i file `farm_1.any` includono la `farm_5.any` configurazione della configurazione delle aziende da uno a cinque, potete includerli come segue:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Utilizzo delle variabili d&#39;ambiente {#using-environment-variables}

È possibile utilizzare variabili d&#39;ambiente in proprietà con valore stringa nel dispatcher. qualsiasi file, anziché codificare i valori. Per includere il valore di una variabile di ambiente, utilizzate il formato `${variable_name}`.

Ad esempio, se il dispatcher. qualsiasi file si trova nella stessa directory della directory della cache, è possibile utilizzare il seguente valore per la [proprietà docroot](dispatcher-configuration.md#main-pars-title-30) :

```xml
/docroot "${PWD}/cache"
```

Ad esempio, se crei una variabile di ambiente denominata `PUBLISH_IP` in cui viene memorizzato il nome host dell&#39;istanza di pubblicazione AEM, è possibile utilizzare la seguente configurazione della proprietà [/renders](dispatcher-configuration.md#main-pars-127-25-0008) :

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Denominazione dell&#39;istanza di dispatcher {#naming-the-dispatcher-instance-name}

Utilizzate la `/name` proprietà per specificare un nome univoco per identificare l&#39;istanza di Dispatcher. La `/name` proprietà è una proprietà di livello principale nella struttura di configurazione.

## Definizione delle industrie agricole {#defining-farms-farms}

La `/farms` proprietà definisce uno o più set di comportamenti Dispatcher, dove ogni set è associato a siti Web o URL diversi. La `/farms` proprietà può includere una sola fattoria o più aziende:

* Utilizzate una sola fattoria per gestire tutte le pagine Web e i siti Web allo stesso modo.
* Create più aziende quando diverse aree del sito Web o siti Web diversi richiedono un comportamento di Dispatcher diverso.

La `/farms` proprietà è una proprietà di livello principale nella struttura di configurazione. Per definire una fattoria, aggiungete una proprietà figlia alla `/farms` proprietà. Utilizzate un nome di proprietà che identifica in modo univoco la fattoria nell&#39;istanza di Dispatcher.

La `/*farmname*` proprietà è con più valori e contiene altre proprietà che definiscono il comportamento del dispatcher:

* Gli URL delle pagine a cui si applica la fattoria.
* Uno o più URL del servizio (in genere delle istanze di pubblicazione AEM) da utilizzare per il rendering dei documenti.
* Statistiche da utilizzare per il bilanciamento del carico più renderer di documenti.
* Diversi altri comportamenti, quali i file alla cache e dove.

Il valore può contenere qualsiasi carattere alfanumerico (a-z, 0-9). L&#39;esempio seguente mostra la definizione dello scheletro per due aziende denominate `/daycom` e `/docsdaycom`:

```xml
#name of dispatcher
/name "day sites"

#farms section defines a list of farms or sites
/farms
{
   /daycom
   {
       ...
   }
   /docdaycom
   {
      ...
   }
}
```

>[!NOTE]
>
>Se utilizzate più di una farm farm, l&#39;elenco viene valutato in basso. Ciò è particolarmente rilevante per la definizione [degli ospitanti](dispatcher-configuration.md#main-pars-117-15-0006) virtuali per i siti Web.

Ogni proprietà farm può contenere le seguenti proprietà secondarie:

| Nome proprietà | Descrizione |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | Pagina principale predefinita (facoltativo) (solo IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | Le intestazioni dalla richiesta HTTP client da passare. |
| [/virtualhosts](#identifying-virtual-hosts-virtual-hosts) | Gli ospitanti virtuali della fattoria. |
| [/sessionmanagement](#enabling-secure-sessions-session-management) | Supporto per gestione e autenticazione delle sessioni. |
| [/renders](#defining-page-renderers-renders) | Server che forniscono pagine sottoposte a rendering (in genere istanze di pubblicazione AEM). |
| [/filter](#configuring-access-to-content-filter) | Definisce gli URL a cui il dispatcher consente l&#39;accesso. |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | Configura l&#39;accesso agli URL personalizzati. |
| [/propagateSyndPost](#forwarding-syndication-requests-propagate-syndpost) | Supporto per l&#39;inoltro delle richieste di sindacazione. |
| [/cache](#configuring-the-dispatcher-cache-cache) | Configura il comportamento di caching. |
| [/statistics](#configuring-load-balancing-statistics) | Definizione delle categorie statistiche per il bilanciamento del carico. |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-sticky-connections-for) | Cartella contenente i documenti fissi. |
| [/health_check](#specifying-a-health-check-page) | L&#39;URL da utilizzare per determinare la disponibilità del server. |
| [/retryDelay](#specifying-the-page-retry-delay) | Il ritardo prima di riprovare a eseguire una connessione non riuscita. |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | Sanzioni che interessano le statistiche per il bilanciamento del carico. |
| [/failover](#using-the-fail-over-mechanism) | Invia di nuovo le richieste a diversi rendering quando la richiesta originale non riesce. |

## Specificate una pagina predefinita (solo IIS) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>Il `/homepage`parametro (solo IIS) non funziona più. Dovete invece utilizzare il modulo [Riscrittura URL IIS](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Se utilizzate Apache, usate il `mod_rewrite` modulo. Per informazioni su `mod_rewrite` (ad esempio [, Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)) consultate la documentazione relativa al sito Web Apache. Quando si utilizza `mod_rewrite`, è consigliabile utilizzare il flag** [&#39;passthrough| PT&#39; (passa al gestore successivo)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** per obbligare il motore di riscrittura a impostare il `uri` campo della `request_rec` struttura interna sul valore del `filename` campo.

<!-- 

Comment Type: draft

<p>The optional /homepage parameter specifies the page that Dispatcher returns when a client requests an undeterminable page or file.</p> 
<p>Typically this situation occurs when a user specifies an URL for which neither IIS or AEM provides an automatic redirection target. For example, if the AEM render instance is shut down after the content is cached, the content redirect URL is unavailable.</p> 
<p>The following example configuration displays the <span class="code">index.html</span> page in such circumstances:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /homepage&nbsp;"/index.html" 
</codeblock>

 -->

<!-- 

Comment Type: draft

<p>The <span class="code">/homepage</span> section is located inside the <span class="code">/farms</span> section, for example:<br /> </p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  #name&nbsp;of&nbsp;dispatcher!!discoiqbr!!/name&nbsp;"day&nbsp;sites"!!discoiqbr!!!!discoiqbr!!#farms&nbsp;section&nbsp;defines&nbsp;a&nbsp;list&nbsp;of&nbsp;farms&nbsp;or&nbsp;sites!!discoiqbr!!/farms!!discoiqbr!!{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/daycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/homepage&nbsp;"/index.html"!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!&nbsp;&nbsp;&nbsp;/docdaycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!} 
</codeblock>

 -->

## Specifica delle intestazioni HTTP per passare attraverso {#specifying-the-http-headers-to-pass-through-clientheaders}

La `/clientheaders` proprietà definisce un elenco delle intestazioni HTTP passate dalla richiesta HTTP client al renderer (istanza AEM).

Per impostazione predefinita, il dispatcher ha inoltrato le intestazioni HTTP standard all&#39;istanza AEM. In alcuni casi, è possibile che vengano inoltrate intestazioni aggiuntive o che vengano rimosse intestazioni specifiche:

* Aggiungete intestazioni, come intestazioni personalizzate, che l&#39;istanza AEM prevede nella richiesta HTTP.
* Rimuovere intestazioni, quali intestazioni di autenticazione, pertinenti solo al server Web.

Se personalizzate il set di intestazioni da superare, dovete specificare un elenco completo delle intestazioni, compresi quelli normalmente inclusi per impostazione predefinita.

Ad esempio, un&#39;istanza di Dispatcher che gestisce le richieste di attivazione delle pagine per le istanze di pubblicazione richiede l&#39; `PATH` intestazione nella `/clientheaders` sezione. L&#39; `PATH` intestazione consente la comunicazione tra l&#39;agente di replica e il dispatcher.

Il codice seguente è una configurazione di esempio per `/clientheaders`:

```shell
/clientheaders
  {
  "CSRF-Token"
  "X-Forwarded-Proto"
  "referer"
  "user-agent"
  "authorization"
  "from"
  "content-type"
  "content-length"
  "accept-charset"
  "accept-encoding"
  "accept-language"
  "accept"
  "host"
  "if-match"
  "if-none-match"
  "if-range"
  "if-unmodified-since"
  "max-forwards"
  "proxy-authorization"
  "proxy-connection"
  "range"
  "cookie"
  "cq-action"
  "cq-handle"
  "handle"
  "action"
  "cqstats"
  "depth"
  "translate"
  "expires"
  "date"
  "dav"
  "ms-author-via"
  "if"
  "lock-token"
  "x-expected-entity-length"
  "destination"
  "PATH"
  }
```

## Identificazione degli ospitanti virtuali {#identifying-virtual-hosts-virtualhosts}

La `/virtualhosts` proprietà definisce un elenco di tutte le combinazioni host/URI accettate da Dispatcher per questa fattoria. Potete usare il carattere asterisco (&quot; *&quot;) come carattere jolly. I valori per la proprietà/ `virtualhosts` proprietà utilizzano il formato seguente:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Facoltativo) `https://` O `https://.`
* `host`: Il nome o l&#39;indirizzo IP del computer host e il numero di porta, se necessario. (Vedete [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23)
* `uri`: (Facoltativo) Il percorso delle risorse.

La seguente configurazione di esempio gestisce le richieste per i domini.com e. ch di mycompany e tutti i domini di mysubdivision:

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

La seguente configurazione gestisce *tutte* le richieste:

```xml
   /virtualhosts
    {
    "*"
    }
```

### Risoluzione dell&#39;host virtuale {#resolving-the-virtual-host}

Quando il dispatcher riceve una richiesta HTTP o HTTPS, trova il valore host virtuale che corrisponde meglio alle intestazioni `host,``uri`e `scheme` alle intestazioni della richiesta. Il dispatcher valuta i valori delle `virtualhosts` proprietà nel seguente ordine:

* Il dispatcher inizia dalla fattoria più bassa e avanza verso l&#39;alto nel dispatcher. any file.
* Per ogni fattoria, il dispatcher inizia con il valore superiore nella `virtualhosts` proprietà e avanza lungo l&#39;elenco di valori.

Il dispatcher trova il valore host virtuale che corrisponde meglio nel modo seguente:

* Il primo host virtuale che corrisponde a tutti e tre i `host`casi, the `scheme`, and the `uri` request is used.
* Se non `virtualhosts` sono `scheme` presenti valori e `uri` parti che corrispondono a `scheme` entrambe e `uri` della richiesta, viene utilizzato il primo host virtuale che corrisponde alla `host` richiesta.
* Se nessun `virtualhosts` valore ha una parte host che corrisponde all&#39;host della richiesta, viene utilizzato l&#39;host virtuale della prima azienda.

Pertanto, posizionate l&#39;host virtuale predefinito nella parte superiore della `virtualhosts` proprietà nella prima fattoria del dispatcher. any.

### Esempio di risoluzione host virtuale {#example-virtual-host-resolution}

L&#39;esempio seguente rappresenta uno snippet da un dispatcher. qualsiasi file che definisce due allevamenti di Dispatcher e ogni fattoria definisce una `virtualhosts` proprietà.

```xml
/farms
  {
  /myProducts 
    { 
    /virtualhosts
      {
      "www.mycompany.com"
      }
    /renders
      {
      /hostname "server1.myCompany.com"
      /port "80"
      }
    }
  /myCompany 
    { 
    /virtualhosts
      {
      "www.mycompany.com/products/*"
      }
    /renders
      {
      /hostname "server2.myCompany.com"
      /port "80"
      }
    }
  }
```

Con questo esempio, la tabella seguente mostra gli host virtuali risolti per le richieste HTTP date:

| URL richiesta | Host virtuale risolto |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/*;` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## Abilitazione delle sessioni sicure - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized`**per** abilitare questa funzione deve essere impostato `"0"` nella `/cache` sezione.

Create una sessione protetta per accedere alla farm farm, in modo che gli utenti debbano accedere per accedere a qualsiasi pagina della fattoria. Dopo aver eseguito l&#39;accesso, gli utenti possono accedere a tutte le pagine della fattoria. Consultate [Creazione di un gruppo](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) utenti chiuso per informazioni sull&#39;utilizzo di questa funzione con i relativi CUG.

La `/sessionmanagement` proprietà è una sottoproprietà di `/farms`.

>[!CAUTION]
>
>Se le sezioni del sito Web usano requisiti di accesso diversi, è necessario definire più aziende.

**/sessionmanagement** presenta diversi parametri secondari:

**/directory** (obbligatorio)

La directory in cui memorizzate le informazioni della sessione. Se la directory non esiste, viene creata.

**/encode** (facoltativo)

Modalità di codifica delle informazioni della sessione. Utilizzare &quot;md 5&quot; per la cifratura utilizzando l&#39;algoritmo md 5 o &quot;hex&quot; per la codifica esadecimale. Se si cifrano i dati della sessione, un utente con accesso al file system non può leggere il contenuto della sessione. Il valore predefinito è &quot;md 5&quot;.

**/header** (facoltativo)

Nome dell&#39;intestazione o del cookie HTTP che memorizza le informazioni sull&#39;autorizzazione. Se archiviate le informazioni nell&#39;intestazione http, utilizzate `HTTP:<*header-name*>`. Per memorizzare le informazioni in un cookie, utilizzate `Cookie:<header-name>`. Se non si specifica un valore `HTTP:authorization` , viene utilizzato.

**/timeout** (facoltativo)

Il numero di secondi per cui la sessione è scaduta dopo che è stata utilizzata per ultimo. Se non viene specificato &quot;800&quot;, la sessione scade oltre 13 minuti dopo l&#39;ultima richiesta dell&#39;utente.

Esempio di configurazione:

```xml
/sessionmanagement 
  { 
  /directory "/usr/local/apache/.sessions" 
  /encode "md5" 
  /header "HTTP:authorization" 
  /timeout "800" 
  }
```

## Definizione dei renderer delle pagine {#defining-page-renderers-renders}

La proprietà /renders definisce l&#39;URL a cui Dispatcher invia le richieste per il rendering di un documento. La sezione di esempio `/renders` seguente identifica una singola istanza AEM per il rendering:

```xml
/renders
  {
    /myRenderer
      {
      # hostname or IP of the renderer
      /hostname "aem.myCompany.com"
      # port of the renderer
      /port "4503"
      # connection timeout in milliseconds, "0" (default) waits indefinitely
      /timeout "0"
      }
  }
```

La sezione /renders di seguito identifica un&#39;istanza AEM che viene eseguita sullo stesso computer di Dispatcher:

```xml
/renders
  {
    /myRenderer
     {
     /hostname "127.0.0.1"
     /port "4503"
     }
  }
```

La sezione /renders di seguito distribuisce le richieste di rendering in modo uniforme tra due istanze AEM:

```xml
/renders
  {
    /myFirstRenderer
      {
      /hostname "aem.myCompany.com"
      /port "4503"
      }
    /mySecondRenderer
      {
      /hostname "127.0.0.1"
      /port "4503"
      }
  }
```

### Opzioni Rendering {#renders-options}

**/timeout**

Specifica il timeout della connessione che accede all&#39;istanza AEM in millisecondi. Il valore predefinito è &quot;0&quot;, comportando l&#39;attesa a tempo indefinito del dispatcher.

**/receiveTimeout**

Specifica il tempo in millisecondi consentito per la risposta. Il valore predefinito è &quot;600000&quot;, che causava l&#39;attesa di 10 minuti da parte del dispatcher. Un&#39;impostazione di &quot;0&quot; elimina completamente il timeout.\
Se viene raggiunto il timeout durante l&#39;analisi delle intestazioni di risposta, viene restituito uno stato HTTP di 504 (Gateway errato). Se il timeout viene raggiunto durante la lettura del corpo della risposta, il dispatcher restituirà la risposta incompleta al client, ma elimina eventuali file della cache che potrebbero essere stati scritti.

**/ipv4**

Specifica se il dispatcher usa la `getaddrinfo` funzione (per ipv 6) o `gethostbyname` la funzione (per ipv 4) per ottenere l&#39;indirizzo IP del rendering. Viene utilizzato `getaddrinfo` un valore pari a 0. Viene utilizzato `gethostbyname` un valore pari a 1. Il valore predefinito è 0.

La funzione getaddrinfo restituisce un elenco di indirizzi IP. Il dispatcher ripeterà l&#39;elenco degli indirizzi finché non stabilisce una connessione TCP/IP. Pertanto, la proprietà ipv 4 è importante quando il nome host di rendering è associato a indirizzi IP con più stati e l&#39;ospitante, in risposta alla funzione getaddrinfo, restituisce un elenco di indirizzi IP sempre nello stesso ordine. In questa situazione, è necessario utilizzare la funzione gethostbyname in modo che l&#39;indirizzo IP con cui si collega il dispatcher sia casuale.

Il bilanciamento del carico di Amazon Elastic (ELB) è un servizio tale che risponde a getaddrinfo con un elenco possibile di indirizzi IP ordinato.

**/secure**

Se la `/secure` proprietà ha un valore di &quot;1&quot; dispatcher usa HTTPS per comunicare con l&#39;istanza AEM. Per ulteriori dettagli, consultate [Configurazione del dispatcher per utilizzare SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Con la versione **4.1.6 di Dispatcher**, puoi configurare la `/always-resolve` proprietà come segue:

* Se impostato su &quot;1&quot;, viene risolto il nome host su ogni richiesta (il dispatcher non memorizzerà mai l&#39;indirizzo IP). Potrebbe verificarsi un leggero impatto sulle prestazioni a causa della chiamata aggiuntiva necessaria per ottenere le informazioni relative all&#39;host per ogni richiesta.
* Se la proprietà non è impostata, l&#39;indirizzo IP verrà memorizzato nella cache per impostazione predefinita.

Inoltre, questa proprietà può essere utilizzata in caso di problemi di risoluzione IP dinamici, come illustrato nel seguente esempio:

```xml
/rend {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## Configurazione dell&#39;accesso al contenuto {#configuring-access-to-content-filter}

Utilizzate `/filter` la sezione per specificare le richieste HTTP accettate dal dispatcher. Tutte le altre richieste vengono inviate al server Web con un codice di errore 404 (pagina non trovata). Se non `/filter` esiste una sezione, tutte le richieste vengono accettate.

**Note:** Requests for the [statfile](dispatcher-configuration.md#main-pars-title-28) are always rejected.

>[!CAUTION]
>
>Per [ulteriori considerazioni sull&#39;accesso tramite Dispatcher, consultate Elenco di controllo](security-checklist.md) della sicurezza del dispatcher. Inoltre, leggi [AEM Security Cheklist](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) per ulteriori dettagli sulla sicurezza relativi all&#39;installazione di AEM.

La sezione /filter consiste in una serie di regole che possono negare o consentire l&#39;accesso al contenuto in base ai pattern nella parte richiesta della richiesta HTTP. Utilizzate una strategia meteortelist per la sezione /filter:

* Innanzitutto, nega l&#39;accesso a tutti gli elementi.
* Consente l&#39;accesso al contenuto in base alle esigenze.

### Definizione di un filtro {#defining-a-filter}

Ogni elemento della `/filter` sezione include un tipo e un pattern associato a un elemento specifico della riga di richiesta o all&#39;intera riga di richiesta. Ciascun filtro può contenere i seguenti elementi:

* **Tipo**: Indica `/type` se consentire o negare l&#39;accesso per le richieste che corrispondono al pattern. Il valore può essere `allow` o `deny`.

* **Elemento della riga richiesta:** Includete `/method``/url`, `/query`o `/protocol` e un pattern per le richieste di filtraggio in base a queste parti specifiche della porzione di richiesta della richiesta HTTP. Il filtro sugli elementi della riga di richiesta (anziché sull&#39;intera riga di richiesta) è il metodo di filtro preferito.

* **proprietà glob**: La `/glob` proprietà viene utilizzata per corrispondere all&#39;intera riga di richiesta della richiesta HTTP.

Per informazioni sulle /glob à, vedere [Progettazione di pattern per le proprietà di glob](#designing-patterns-for-glob-properties). Le regole per l&#39;uso dei caratteri jolly in /glob à sono valide anche per gli elementi per gli elementi corrispondenti della riga di richiesta.

>[!NOTE]
>
>Nella versione 4.2.0 di Dispatcher sono stati aggiunti diversi miglioramenti per le configurazioni di filtro e le funzionalità di registrazione:
>
>* [Supporto per espressioni regolari POSIX](dispatcher-configuration.md#main-pars-title-1996763852)
>* [Supporto per filtrare altri elementi dell&#39;URL della richiesta](dispatcher-configuration.md#main-pars-title-694578373)
>* [Registrazione registrazione](dispatcher-configuration.md#main-pars-title-1950006642)
>



#### The request-line Part of HTTP Requests {#the-request-line-part-of-http-requests}

HTTP/1.1 definisce la [riga di richiesta](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) come segue:

*Request Request-URI HTTP-Version*&lt; CRLF &gt;

I caratteri &lt; CRLF &gt; riportavano un ritorno a capo seguito da un feed di riga. L&#39;esempio seguente è la riga di richiesta che viene recisa quando un client richiede la pagina en del sito Geometrixx-Outoors:

GET /content/geometrixx-outdoors/en.html HTTP .1 .1 &lt; CRLF &gt;

I pattern devono prendere in considerazione gli spazi nella riga di richiesta e i caratteri &lt; CRLF &gt;.

#### Filtro esempio: Rifiuta tutto {#example-filter-deny-all}

La sezione di filtro seguente fa sì che il dispatcher neghi le richieste di tutti i file. È consigliabile negare l&#39;accesso a tutti i file e consentire l&#39;accesso a specifiche aree.

```xml
  /0001  { /glob "*" /type "deny" }
```

Le richieste inviate a un&#39;area rifiutata in modo esplicito producono un codice di errore 404 (pagina non trovata) restituita.

#### Filtro esempio: Rifiuta acess a aree specifiche {#example-filter-deny-acess-to-specific-areas}

I filtri consentono inoltre di negare l&#39;accesso a vari elementi, ad esempio pagine ASP e aree sensibili all&#39;interno di un&#39;istanza di pubblicazione. Il filtro seguente nega l&#39;accesso alle pagine ASP:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Filtro esempio: Abilita richieste POST {#example-filter-enable-post-requests}

Il filtro di esempio seguente consente di inviare i dati del modulo tramite il metodo POST:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Filtro esempio: Consenti accesso alla console flusso di lavoro {#example-filter-allow-access-to-the-workflow-console}

L&#39;esempio seguente mostra un filtro utilizzato per negare l&#39;accesso esterno alla console Workflow:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

Se l&#39;istanza di pubblicazione utilizza un contesto applicazione Web (ad esempio pubblicazione), può essere aggiunto anche alla definizione del filtro.

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

Se è necessario accedere a singole pagine all&#39;interno dell&#39;area limitata, è possibile consentirne l&#39;accesso. Ad esempio, per consentire l&#39;accesso alla scheda Archivio nella console Flusso di lavoro, aggiungi la sezione seguente:

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>Quando si applicano più pattern di filtri a una richiesta, l&#39;ultimo pattern di filtro applicato è efficace.

#### Filtro esempio: Utilizzo delle espressioni regolari {#example-filter-using-regular-expressions}

Questo filtro abilita le estensioni nelle directory di contenuto non pubbliche utilizzando un&#39;espressione regolare, definita qui tra virgolette singole:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Filtro esempio: Filtrare altri elementi di un URL di richiesta {#example-filter-filter-additional-elements-of-a-request-url}

Uno dei miglioramenti introdotti nel dispatcher 4.2.0 è la capacità di filtrare altri elementi dell&#39;URL della richiesta. I nuovi elementi introdotti sono:

* path
* selectors
* extension
* suffix

Questi possono essere configurati aggiungendo la proprietà dello stesso nome alla regola di filtro: `/path`, `/selectors`e `/extension``/suffix` rispettivamente.

Di seguito è riportato un esempio di regole che blocca l&#39;acquisizione dei contenuti dal `/content` percorso e dalla relativa sottostruttura, utilizzando i filtri per percorso, selettori ed estensioni:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### Esempio /filter sezione {#example-filter-section}

Durante la configurazione di Dispatcher, è necessario limitare il più possibile l&#39;accesso esterno. L&#39;esempio seguente fornisce un accesso minimo ai visitatori esterni:

* `/content`
* contenuti vari quali design e librerie client; ad esempio:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

Dopo aver creato i filtri, [verificate l&#39;accesso alla pagina](dispatcher-configuration.md#main-pars-title-19) per assicurarvi che l&#39;istanza AEM sia protetta.

La sezione /filter seguente del dispatcher. Qualsiasi file può essere utilizzato come base nel file [di configurazione](dispatcher-configuration.md) del dispatcher.

Questo esempio si basa sul file di configurazione predefinito fornito con Dispatcher ed è inteso come esempio da utilizzare in un ambiente di produzione. Gli elementi con il prefisso # sono disattivati (commentato), se decidete di attivarli (rimuovendo il # su quella riga) in modo che possano avere un impatto sulla sicurezza.

È consigliabile negare l&#39;accesso a tutti gli elementi, e consentire l&#39;accesso a elementi specifici (limitati):

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:32:37.986-0400

<p>We should mention the config files that are shipped with the dispatcher distribution and only give a few examples here. This aims to avoid confusion and reduce content maintenance.<br /> </p>

 -->

```xml
  /filter
      {
      # Deny everything first and then allow specific entries
      /0001 { /type "deny" /glob "*" }
      
      # Open consoles
#     /0011 { /type "allow" /url "/admin/*"  }  # allow servlet engine admin
#     /0012 { /type "allow" /url "/crx/*"    }  # allow content repository
#     /0013 { /type "allow" /url "/system/*" }  # allow OSGi console
        
      # Allow non-public content directories
#     /0021 { /type "allow" /url "/apps/*"   }  # allow apps access
#     /0022 { /type "allow" /url "/bin/*"    }
      /0023 { /type "allow" /url "/content*" }  # disable this rule to allow mapped content only
      
#     /0024 { /type "allow" /url "/libs/*"   }
#     /0025 { /type "deny"  /url "/libs/shindig/proxy*" } # if you enable /libs close access to proxy

#     /0026 { /type "allow" /url "/home/*"   }
#     /0027 { /type "allow" /url "/tmp/*"    }
#     /0028 { /type "allow" /url "/var/*"    }

      # Enable extensions in non-public content directories, using a regular expression
      /0041
        {
        /type "allow"
        /extension '(css|gif|ico|js|png|swf|jpe?g)'
        }

      # Enable features 
      /0062 { /type "allow" /url "/libs/cq/personalization/*"  }  # enable personalization

      # Deny content grabbing, on all accessible pages, using regular expressions
      /0081
        {
        /type "deny"
        /selectors '((sys|doc)view|query|[0-9-]+)'
        /extension '(json|xml)'
        }
      # Deny content grabbing for /content and its subtree
      /0082
        {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }

#     /0087 { /type "allow" /method "GET" /extension 'json' "*.1.json" }  # allow one-level json requests
}
```

>[!NOTE]
>
>Se utilizzato con Apache, progettate i pattern URL del filtro in base alla proprietà dispatcheruseprocessedurl del modulo Dispatcher. (consultate [Apache Web Server - Configurare il server Apache Web per Dispatcher](dispatcher-install.md#main-pars-55-35-1022).)

>[!NOTE]
>
>I filtri 0030 e 0031 relativi ai contenuti multimediali dinamici sono applicabili a AEM 6.0 e versioni successive.

Se scegliete di estendere l&#39;accesso, prendete in considerazione le seguenti raccomandazioni:

* L&#39;accesso esterno deve `/admin` essere sempre disattivato *completamente* se si utilizza CQ versione 5.4 o una versione precedente.

* Care must be taken when allowing access to files in `/libs`. L&#39;accesso deve essere consentito su base individuale.
* Rifiuta l&#39;accesso alla configurazione di replica in modo che non possa essere visualizzato:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Rifiuta l&#39;accesso al proxy inverso Google Gadgets:

   * `/libs/opensocial/proxy*`

A seconda dell&#39;installazione, possono essere disponibili risorse aggiuntive in, `/libs``/apps` o altrove, che devono essere rese disponibili. Potete utilizzare il `access.log` file come metodo per determinare le risorse a cui si accede esternamente.

>[!CAUTION]
>
>L&#39;accesso alle console e alle directory può presentare un rischio di protezione per gli ambienti di produzione. A meno che non abbiano giustificazione esplicita, dovrebbero rimanere disattivate (commentato).

>[!CAUTION]
>
>Se [utilizzate i rapporti in un ambiente](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) di pubblicazione, configurate Dispatcher per negare l&#39;accesso `/etc/reports` ai visitatori esterni.

### Limitazione delle stringhe query {#restricting-query-strings}

Dalla versione 4.1.5 di Dispatcher, utilizzare la `/filter` sezione per limitare le stringhe di query. Si consiglia vivamente di consentire le stringhe di query ed escludere la quota generica tramite gli elementi `allow` del filtro.

Una singola voce può avere *una descrizione* o una combinazione di *metodo*,*url*,*query* e *versione* , ma non entrambe. L&#39;esempio seguente consente la stringa `a=*` di query e denota tutte le altre stringhe di query per gli URL che risolvono il `/etc` nodo:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Se una regola contiene, `/query`corrisponderà solo alle richieste che contengono una stringa di query e corrispondono al pattern di query fornito.
>
>Nell&#39;esempio precedente, se le richieste a `/etc` cui non è presente alcuna stringa query non sono consentite, sarà necessario disporre delle seguenti regole:


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Verifica della sicurezza del dispatcher {#testing-dispatcher-security}

I filtri di dispatcher bloccano l&#39;accesso alle pagine e agli script seguenti nelle istanze di pubblicazione AEM. Utilizzate un browser Web per tentare di aprire le seguenti pagine come desiderate un visitatore di un sito e verificare che sia restituito un codice 404. Se si ottengono altri risultati, regolate i filtri.

È necessario visualizzare il rendering normale della pagina per /content/add_valid_page.html? debug = layout.


* /admin
* /system/console
* /dav/crx.default
* /crx
* /bin/crxde/logs
* /jcr: system/jcr: Versionstorage. json
* /_jcr_system/_jcr_versionStorage.json
* /libs/wcm/core/content/siteadmin.html
* /libs/collab/core/content/admin.html
* /libs/cq/ui/content/dumplibs.html
* /var/linkchecker.html
* /etc/linkchecker.html
* /home/users/a/admin/profile.json
* /home/users/a/admin/profile.xml
* /libs/cq/core/content/login.json
* /content/../libs/foundation/components/text/text.jsp
* /content/.{.}/libs/foundation/components/text/text. jsp
* /apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr % 3 acontent/jcr % 3 adata
* /libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet
* /content.pages.json
* /content.languages.json
* /content.blueprint.json
* /content.-1. json
* /content.10.json
* /content.infinity.json
* /content.tidy.json
* /content.tidy.-1. blubber. json
* /content/dam.tidy.-100. json
* /content/content/geometrixx.sitemap.txt
* /content/add_valid_page.query.json? statement =//*
* /content/add_valid_page.qu % 65 ry. js % 6 Fn? statement =//*
* /content/add_valid_page.query.json? statement =//*[@ transportpassword]/(@ transportpassword % 20|% 20@transportUri % 20|% 20@transportUser)
* /content/add_valid_path_to_a_page/_jcr_content.json
* /content/add_valid_path_to_a_page/jcr: content. json
* /content/add_valid_path_to_a_page/_jcr_content.feed
* /content/add_valid_path_to_a_page/jcr: content. feed
* /content/add_valid_path_to_a_page/pagename._ jcr_ content. feed
* /content/add_valid_path_to_a_page/pagename.jcr: content. feed
* /content/add_valid_path_to_a_page/pagename.docview.xml
* /content/add_valid_path_to_a_page/pagename.docview.json
* /content/add_valid_path_to_a_page/pagename.sysview.xml
* /etc.xml
* /content.feed.xml
* /content.rss.xml
* /content.feed.html
* /content/add_valid_page.html? debug = layout
* /projects
* /tagging
* /etc/replication.html
* /etc/cloudservices.html
* /benvenuti

Eseguite il comando seguente in un prompt di terminale o comandi per determinare se l&#39;accesso alla scrittura anonimo è abilitato. Non è possibile scrivere dati sul nodo.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Eseguite il seguente comando in un prompt di terminale o comandi per tentare di annullare la validità della cache del dispatcher, e assicuratevi di ricevere una risposta codice 404:

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## Abilitazione dell&#39;accesso agli URL personalizzati {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The com.adobe.granite.dispatcher.vanityurl.content package needs to be made public before publishing this contnet.</p>

 -->

Configurare il dispatcher per abilitare l&#39;accesso agli URL personalizzati configurati per le pagine CQ o AEM.

Quando l&#39;accesso agli URL personalizzati è abilitato, Dispatcher chiama periodicamente un servizio che viene eseguito sull&#39;istanza di rendering per ottenere un elenco di URL personalizzati. Dispatcher memorizza questo elenco in un file locale. Quando una richiesta di pagina viene rifiutata a causa di un filtro nella `/filter` sezione, il dispatcher organizza l&#39;elenco degli URL personalizzati. Se l&#39;URL rifiutato è presente nell&#39;elenco, il dispatcher consente l&#39;accesso all&#39;URL personalizzato.

Per abilitare l&#39;accesso agli URL personalizzati, aggiungete una `/vanity_urls` sezione alla `/farms` sezione, simile al seguente esempio:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` La sezione contiene le proprietà seguenti:

* `/url`: Percorso del servizio URL personalizzato che viene eseguito nell&#39;istanza di rendering. Il valore di questa proprietà `"/libs/granite/dispatcher/content/vanityUrls.html"`deve essere.

* `/file`: Percorso del file locale in cui Dispatcher memorizza l&#39;elenco degli URL personalizzati. Assicuratevi che il dispatcher abbia accesso in scrittura a questo file.
* `/delay`: (Secondi) L&#39;ora tra chiamate al servizio URL personalizzato.

>[!NOTE]
>
>Se il rendering è un&#39;istanza di AEM, è necessario installare il [pacchetto vanityurls-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) per installare il servizio URL personalizzato. (consultate [Accedere a Package Share](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare).)

Utilizzate la seguente procedura per abilitare l&#39;accesso agli URL personalizzati.

1. Se il servizio di rendering è un&#39;istanza di AEM, installate il pacchetto com. adobe. granite. dispatcher. vanityurl. content nell&#39;istanza di pubblicazione (consultate la nota sopra).
1. Per ogni URL personalizzato configurato per una pagina AEM o CQ, assicurati che ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` la configurazione neghi l&#39;URL. Se necessario, aggiungete un filtro che nega l&#39;URL.
1. Aggiungete la `/vanity_urls` sezione di seguito `/farms`.
1. Riavviate Apache Web Server.

## Richieste di sindacazione - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

Le richieste di sindacazione sono solitamente destinate esclusivamente al dispatcher, quindi per impostazione predefinita non vengono inviate al renderer (ad esempio, un&#39;istanza di AEM).

Se necessario, impostate la proprietà /propagateSyndPost su &quot;1&quot; per inoltrare le richieste di sindacazione al dispatcher. Se impostata, dovete accertarvi che le richieste POST non vengano rifiutate nella sezione del filtro.

## Configurazione della cache del dispatcher - /cache {#configuring-the-dispatcher-cache-cache}

La `/cache` sezione controlla in che modo il dispatcher memorizza nella cache i documenti. Configurate diverse sottoproprietà per implementare le strategie di caching:


* /docroot
* /statfile
* /serveStaleOnError
* /allowAuthorized
* /regole
* /statfileslevel
* /invalidate
* /invalidateHandler
* /allowedClients
* /ignoreUrlParams
* /headers
* /mode
* /gracePeriod


Esempio di sezione della cache di esempio:

```xml
/cache
  {
  /docroot "/opt/dispatcher/cache"
  /statfile  "/tmp/dispatcher-website.stat"          
  /allowAuthorized "0"
      
  /rules
    {
    # List of files that are cached
    }

  /invalidate
    {
    # List of files that are auto-invalidated
    }
  }
  
```

>[!NOTE]
>
>Per il caching sensibile alle autorizzazioni, leggete [la cache Contenuto protetto](permissions-cache.md).

### Specifica della directory cache {#specifying-the-cache-directory}

La `/docroot` proprietà identifica la directory in cui vengono memorizzati i file memorizzati nella cache.

>[!NOTE]
>
>Il valore deve essere lo stesso percorso della radice del documento del server Web in modo che il dispatcher e il server Web gestiscono gli stessi file.\
>Il server Web è responsabile della distribuzione del codice di stato corretto quando viene utilizzato il file della cache del dispatcher, per questo è importante che sia accessibile anche.

Se utilizzate più aziende, ogni fattoria deve utilizzare un&#39;altra radice documento.

### Denominazione del file statico {#naming-the-statfile}

La `/statfile` proprietà identifica il file da utilizzare come file statico. Il dispatcher usa questo file per registrare l&#39;ora dell&#39;aggiornamento del contenuto più recente. Il file statico può essere qualsiasi file presente sul server Web.

Lo statfile non ha alcun contenuto. Quando il contenuto viene aggiornato, il dispatcher aggiorna la marca temporale. Il file statico predefinito è denominato. stat e viene memorizzato nel docroot. Il dispatcher blocca l&#39;accesso al statfile statico.

>[!NOTE]
>
>Se `/statfileslevel` è configurato, Dispatcher ignora la `/statfile` proprietà e utilizza. stat come nome.

### Trasmissione di documenti Stalla quando si verificano errori {#serving-stale-documents-when-errors-occur}

La `/serveStaleOnError` proprietà controlla se il dispatcher restituisce documenti invalidati quando il server di rendering restituisce un errore. Per impostazione predefinita, quando un statfile statico viene toccato e invalida il contenuto memorizzato nella cache, il contenuto memorizzato nella cache viene eliminato dal dispatcher alla sua successiva richiesta.

Se `/serveStaleOnError` è impostato su &quot;1&quot;, il dispatcher non elimina il contenuto invalidato dalla cache a meno che il server di rendering non restituisca una risposta corretta. Una risposta di 5 xx da AEM o un timeout di connessione causa la trasmissione del contenuto non aggiornato e la risposta con lo stato HTTP di 111 (convalida non riuscita).

### Memorizzazione nella cache quando viene utilizzata l&#39;autenticazione {#caching-when-authentication-is-used}

La `/allowAuthorized` proprietà controlla se le richieste contenenti le seguenti informazioni di autenticazione sono memorizzate nella cache:

* L&#39; `authorization` intestazione.
* Un cookie denominato `authorization`.
* Un cookie denominato `login-token`.

Per impostazione predefinita, le richieste che includono queste informazioni di autenticazione non vengono memorizzate nella cache perché l&#39;autenticazione non viene eseguita quando viene restituito un documento nella cache al client. Questa configurazione impedisce al dispatcher di distribuire i documenti memorizzati nella cache agli utenti che non dispongono dei diritti necessari.

Tuttavia, se i vostri requisiti consentono la memorizzazione nella cache dei documenti autenticati, impostate /allowAuthorized su uno:

`/allowAuthorized "1"`

>[!NOTE]
>
>Per abilitare la gestione delle sessioni (utilizzando la `/sessionmanagement` proprietà), è `/allowAuthorized` necessario impostare la `"0"`proprietà.

### Specifica dei documenti nella cache {#specifying-the-documents-to-cache}

La `/rules` proprietà controlla i documenti memorizzati nella cache in base al percorso del documento. A prescindere dalla proprietà /rules, Dispatcher non memorizza mai nella cache un documento nelle seguenti circostanze:

* Se l&#39;URI di richiesta contiene un punto interrogativo (&quot;? &quot;).\
   In genere indica una pagina dinamica, ad esempio un risultato di ricerca che non deve essere memorizzato nella cache.
* L&#39;estensione del file è mancante.\
   Il server Web richiede l&#39;estensione per determinare il tipo di documento (tipo MIME).
* L&#39;intestazione di autenticazione è impostata (può essere configurata)
* Se l&#39;istanza AEM risponde con le seguenti intestazioni:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>I metodi GET o HEAD (per i metodi dell&#39;intestazione HTTP) possono essere memorizzati nella cache dal dispatcher. Per ulteriori informazioni sul caching delle intestazioni della risposta, consultate la [sezione Intestazioni](dispatcher-configuration.md#caching-http-response-headers) risposta HTTP.

Ogni elemento della proprietà /rules include un [pattern di glob](#designing-patterns-for-glob-properties) e un tipo:

* Il pattern di glob viene utilizzato per far corrispondere il percorso del documento.
* Il tipo indica se memorizzare nella cache i documenti che corrispondono al pattern della glob. Il valore può essere consentito (per memorizzare nella cache il documento) o negare (per eseguire il rendering sempre del documento).

Se non hai pagine dinamiche (oltre quelle già escluse dalle regole elencate), puoi configurare il dispatcher in modo da memorizzare tutti gli elementi nella cache. La sezione delle regole si presenta come segue:

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

Per informazioni sulle proprietà della glob, vedere [Progettazione di pattern per le proprietà di glob](#designing-patterns-for-glob-properties).

Se alcune sezioni della pagina sono dinamiche (ad esempio un&#39;applicazione di notizie) o in un gruppo di utenti chiuso, puoi definire le eccezioni:

>[!NOTE]
>
>I gruppi chiusi di utenti non devono essere memorizzati nella cache in quanto i diritti utente non vengono verificati per le pagine memorizzate nella cache.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }   
  }
```

**Compressione**

Sui server Web Apache potete comprimere i documenti memorizzati nella cache. La compressione consente ad Apache di restituire il documento in un modulo compresso, se richiesto dal client. La compressione viene effettuata automaticamente abilitando il modulo `mod_deflate`Apache, ad esempio:

```xml
AddOutputFilterByType DEFLATE text/plain
```

Il modulo viene installato per impostazione predefinita con Apache 2. x.

<!-- 

Comment Type: draft

<note type="note"> 
 <p>Depending on the Apache web server version you can enable <span class="code">gzip</span> compression as follows:</p> 
 <ul> 
  <li>For Apache 1.3 you can enable the <span class="code">mod_gzip </span>module. The module must be downloaded and installed.</li> 
  <li>For Apache 2.x you can enable the <span class="code">mod_deflate</span> module. The module is installed by default with Apache 2.x.<br /> </li> 
 </ul> 
 <p> </p> 
</note>

 -->

<!-- 

Comment Type: draft

<p>The following rule caches all documents in compressed form; Apache can return either the uncompressed or the compressed form to the client:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /rules!!discoiqbr!!&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/rulelabel&nbsp;&nbsp;{&nbsp;&nbsp;/glob&nbsp;"*"&nbsp;/type&nbsp;"allow"&nbsp;&nbsp;/compress&nbsp;"gzip"&nbsp;}!!discoiqbr!!&nbsp;&nbsp;} 
</codeblock>

 -->

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-11-13T09:23:24.326-0500

<p>Hidden the <span class="code">mod_gzip</span> content as requested in CQDOC-11124.</p>

 -->

### Annullamento della validità dei file per livello di cartella {#invalidating-files-by-folder-level}

Utilizzate la `/statfileslevel` proprietà per annullare la validità dei file nella cache in base al percorso del percorso:

* Il dispatcher crea `.stat`file in ciascuna cartella dalla cartella docroot al livello specificato. La cartella docroot è livello 0.
* I file vengono invalidati toccando il `.stat` file. La data dell&#39;ultima modifica `.stat` del file viene confrontata con l&#39;ultima data di modifica di un documento memorizzato nella cache. Il documento viene nuovamente caricato se il `.stat` file è più recente.

* Quando un file che si trova a un determinato livello viene invalidato, **tutti** `.stat` i file dal docroot **al** livello del file invalidato o alla configurazione `statsfilevel` (più piccola) verranno toccati.

   * Ad esempio: se impostate la `statfileslevel` proprietà su 6 e un file viene invalidato a livello 5, ogni `.stat` file da docroot a 5 viene toccato. Continuate con questo esempio, se un file viene invalidato al livello 7 e quindi ogni. `stat` da docroot a 6 viene toccato (dal momento `/statfileslevel = "6"`).

Sono interessate solo le risorse** lungo il percorso** nel file invalidated. Considerate l&#39;esempio seguente: un sito Web utilizza la struttura `/content/myWebsite/xx/.` Se impostata `statfileslevel` su 3, un `.stat`file viene creato come segue:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

Quando un file viene invalidato `/content/myWebsite/xx` , ogni `.stat` file dal passaggio verso il basso viene `/content/myWebsite/xx`toccato. Ciò si applica solo `/content/myWebsite/xx` ad esempio `/content/myWebsite/yy` , non `/content/anotherWebSite`ad esempio.

>[!NOTE]
>
>È possibile evitare la convalida inviando un&#39;intestazione `CQ-Action-Scope:ResourceOnly`aggiuntiva. Questo può essere utilizzato per svuotare determinate risorse senza invalidare altre parti della cache. Consultate [questa pagina](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) e [Annullamento manuale della cache del dispatcher](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) per ulteriori dettagli.

>[!NOTE]
>
>Se si specifica un valore per la `/statfileslevel` proprietà, la `/statfile` proprietà viene ignorata.

### Annullamento automatico dei file memorizzati nella cache {#automatically-invalidating-cached-files}

La `/invalidate` proprietà definisce i documenti che vengono invalidati automaticamente quando il contenuto viene aggiornato.

Con la convalida automatica, il dispatcher non elimina i file memorizzati nella cache dopo un aggiornamento del contenuto, ma ne verifica la validità al momento della successiva richiesta. I documenti nella cache che non vengono invalidati automaticamente rimarranno nella cache finché non vengono eliminati esplicitamente da un aggiornamento del contenuto.

La convalida automatica viene generalmente utilizzata per le pagine HTML. Le pagine HTML contengono spesso collegamenti verso altre pagine, rendendo difficile determinare se un aggiornamento del contenuto influisce su una pagina. Per verificare che tutte le pagine rilevanti vengano invalidate quando il contenuto viene aggiornato, invalida automaticamente tutte le pagine HTML. La seguente configurazione invalida tutte le pagine HTML:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Per informazioni sulle proprietà della glob, vedere [Progettazione di pattern per le proprietà di glob](#designing-patterns-for-glob-properties).

Questa configurazione causa la seguente attività quando /content/geometrixx/en viene attivato:

* Tutti i file con pattern.* vengono rimossi dalla cartella /content/geometrixx/.
* La cartella /content/geometrixx/en/_jcr_content viene rimossa.
* Tutti gli altri file che corrispondono alla configurazione /invalidate non vengono eliminati immediatamente. Questi file vengono eliminati quando si verifica la richiesta successiva. Nell&#39;esempio /content/geometrixx.html non viene eliminato, verrà eliminato quando /content/geometrixx.html viene richiesto.

Se si offrono file PDF e ZIP generati automaticamente per il download, potrebbe essere necessario annullare automaticamente anche questi. Esempio di configurazione:

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

L&#39;integrazione AEM con Adobe Analytics offre i dati di configurazione in un file analytics. sitecatalyst. js nel sito Web. L&#39;esempio dispatcher. Qualsiasi file fornito con Dispatcher include la seguente regola di annullamento della validità del file:

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### Uso degli script di annullamento di validità {#using-custom-invalidation-scripts}

La proprietà /invalidateHandler consente di definire uno script chiamato per ogni richiesta di annullamento della validità ricevuta dal dispatcher.

Viene richiamata con i seguenti argomenti:

* Punto di controllo\
   Percorso del contenuto invalidato
* Azione\
   Azione di replica (ad esempio Attiva, Disattiva)
* Ambito azione\
   Ambito dell&#39;azione di replica (vuoto, a meno che non venga inviato un titolo, `CQ-Action-Scope: ResourceOnly` consultate [Annullamento della validità delle pagine memorizzate nella cache da AEM](page-invalidate.md) )

Questo può essere utilizzato per proteggere molti casi d&#39;uso diversi, ad esempio la nullità di altre caching specifiche dell&#39;applicazione, o per gestire i casi in cui l&#39;URL esternalizzato di una pagina e la sua posizione nel docroot non corrispondono al percorso del contenuto.

Di seguito viene eseguito il login di script di ogni richiesta invalidata a un file.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### esempio di script del gestore di annullamento validità {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limitazione dei client che possono svuotare la cache {#limiting-the-clients-that-can-flush-the-cache}

La proprietà /allowedClients definisce client specifici ai quali è consentito cancellare la cache. I pattern di visualizzazione vengono associati al IP.

L&#39;esempio seguente:

1. nega l&#39;accesso a qualsiasi client
1. consente di accedere in modo esplicito all&#39;localhost localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

Per informazioni sulle proprietà della glob, vedere [Progettazione di pattern per le proprietà di glob](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>Si consiglia di definire /allowedClients.
>
>In caso contrario, un client può emettere una chiamata per cancellare la cache; se questa operazione viene eseguita ripetutamente, può avere forte impatto sulle prestazioni del sito.

### Ignorare i parametri URL {#ignoring-url-parameters}

`ignoreUrlParams` La sezione definisce i parametri degli URL che vengono ignorati quando si determina se una pagina viene memorizzata nella cache o inviata dalla cache:

* Quando un URL di richiesta contiene parametri che vengono ignorati, la pagina viene memorizzata nella cache.
* Quando un URL di richiesta contiene uno o più parametri non ignorati, la pagina non viene memorizzata nella cache.

Quando un parametro viene ignorato per una pagina, la pagina viene memorizzata nella cache al primo avvio della pagina. Le richieste successive per la pagina vengono servite della pagina memorizzata nella cache, indipendentemente dal valore del parametro nella richiesta.

Per specificare quali parametri vengono ignorati, aggiungete le regole glob alla `ignoreUrlParams` proprietà:

* Per ignorare un parametro, create una proprietà di glob che consenta il parametro.
* Per impedire che la pagina venga memorizzata nella cache, create una proprietà di glob che neghi il parametro.

L&#39;esempio seguente causa la mancata memorizzazione del parametro &quot;q&quot; da parte del dispatcher, in modo che gli URL di richiesta che includono il parametro q siano memorizzati nella cache:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

Utilizzando il `ignoreUrlParams` valore di esempio, la seguente richiesta HTTP causa la memorizzazione nella cache della pagina perché il `q` parametro viene ignorato:

```xml
GET /mypage.html?q=5
```

Utilizzando il `ignoreUrlParams` valore di esempio, la seguente richiesta HTTP causa **la mancata** memorizzazione della pagina nella cache perché il `p` parametro non viene ignorato:

```xml
GET /mypage.html?q=5&p=4
```

Per informazioni sulle proprietà della glob, vedere [Progettazione di pattern per le proprietà di glob](#designing-patterns-for-glob-properties).

### Memorizzazione nella cache delle intestazioni di risposta HTTP {#caching-http-response-headers}

>[!NOTE]
>
>Questa funzione è disponibile con la versione **4.1.11** del dispatcher.

La `/headers` proprietà consente di definire i tipi di intestazione HTTP che verranno memorizzati nella cache dal dispatcher. Sulla prima richiesta a una risorsa non memorizzata nella cache, tutte le intestazioni corrispondenti a uno dei valori configurati (vedere la configurazione di seguito) sono memorizzate in un file separato, accanto al file della cache. Nelle richieste successive alla risorsa memorizzata nella cache, le intestazioni memorizzate vengono aggiunte alla risposta.

Di seguito è riportato un esempio della configurazione predefinita:

```xml
/cache {
  ...
  /headers {
    "Cache-Control"
    "Content-Disposition"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "X-Content-Type-Options"
    "Last-Modified"
  }
}
```

>[!NOTE]
>
>Inoltre, i caratteri glogging dei file non sono consentiti. Per ulteriori dettagli, vedere [Progettazione di pattern per le proprietà di glob](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Se devi memorizzare e distribuire intestazioni di risposta etag da AEM, effettua le seguenti operazioni:
>
>* Aggiungete il nome dell&#39;intestazione nella `/cache/headers`sezione.
>* Aggiungi la seguente direttiva [Apache](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) nella sezione relativa del dispatcher:
>



```xml
FileETag none
```

### Autorizzazioni del file cache del dispatcher {#dispatcher-cache-file-permissions}

La `mode` proprietà specifica quali autorizzazioni file vengono applicate alle nuove directory e ai file nella cache. Questa impostazione è limitata dal `umask` processo di chiamata. Si tratta di un numero ottale costruito partendo dalla somma di uno o più dei seguenti valori:

* 0400 Consente di leggere il proprietario.
* 0200 Consentire la scrittura per proprietario.
* 0100 Consente al proprietario di effettuare ricerche nelle directory.
* 0040 Consente di leggere i membri del gruppo.
* 0020 Consente di scrivere in base ai membri del gruppo.
* 0010 Consente ai membri del gruppo di effettuare ricerche nella directory.
* 0004 Consente di leggere gli altri.
* 0002 Consente di scrivere in scrittura.
* 0001 Consente agli altri di eseguire ricerche nella directory.

Il valore predefinito è 0755, che consente al proprietario di leggere, scrivere o cercare, nonché il gruppo e altri da leggere o cercare.

### Chiusura del file. stat {#throttling-stat-file-touching}

Con la `/invalidate` proprietà predefinita, ogni attivazione invalida efficacemente tutti `.html` i file (quando il percorso corrisponde alla `/invalidate` sezione). Su un sito Web con traffico considerevole, più, le attivazioni successive aumenteranno il carico della CPU sul backend. In questo scenario, sarebbe auspicabile «throttle» `.stat` toccare per mantenere il sito Web reattivo. È possibile farlo utilizzando la `/gracePeriod` proprietà.

La `/gracePeriod` proprietà definisce il numero di secondi in secondi a una stale, la quale potrebbe comunque essere servita dalla cache dopo l&#39;ultima attivazione. La proprietà può essere utilizzata in una configurazione in cui un batch di attivazioni altrimenti invaliderebbe l&#39;intera cache. Il valore consigliato è 2 secondi.

Per ulteriori dettagli, leggere anche le `/invalidate``/statfileslevel`sezioni riportate di seguito.

## Configurazione della validità della cache basata su ora - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

Se impostato, la `enableTTL` proprietà valuta le intestazioni di risposta dal back-end e, se esse contengono una `Cache-Control` data o `Expires` un&#39;età massima, viene creato un file vuoto accanto al file della cache, con il tempo di modifica uguale alla data di scadenza. Quando il file memorizzato nella cache viene richiesto oltre il tempo di modifica, viene richiesto automaticamente dal back-end.

Potete abilitare la funzione aggiungendo questa riga al `dispatcher.any` file:

```xml
/enableTTL "1"
```

>[!NOTE]
>
>Questa funzione è disponibile con la versione **4.1.11** del dispatcher.

## Configurazione del bilanciamento del carico - /statistics {#configuring-load-balancing-statistics}

La `/statistics` sezione definisce le categorie di file per i quali il dispatcher indica la capacità di risposta di ogni rendering. Il dispatcher utilizza i punteggi per determinare quale rendering inviare una richiesta.

Ogni categoria creata definisce un pattern di glob. Il dispatcher confronta l&#39;URI del contenuto richiesto con questi pattern per determinare la categoria del contenuto richiesto:

* L&#39;ordine delle categorie determina l&#39;ordine in cui vengono confrontati con l&#39;URI.
* Il primo pattern di categoria che corrisponde all&#39;URI è la categoria del file. Non vengono valutati più pattern di categoria.

Il dispatcher supporta un massimo di 8 categorie statistiche. Se si definiscono più di 8 categorie, vengono utilizzati solo i primi 8.

**Rendering selezione**

Ogni volta che Dispatcher richiede una pagina sottoposta a rendering, utilizza il seguente algoritmo per selezionare il rendering:

1. Se la richiesta contiene il nome di rendering in un `renderid` cookie, il dispatcher utilizza il rendering.
1. Se la richiesta non include `renderid` cookie, Dispatcher confronta le statistiche di rendering:

   1. Dispatcher determina la richiesta dell&#39;URI della richiesta.
   1. Il dispatcher determina quale rendering ha il punteggio risposta più basso per quella categoria e seleziona il rendering.

1. Se non viene ancora selezionato alcun rendering, utilizzate il primo rendering nell&#39;elenco.

Il punteggio per la categoria di un rendering si basa sui tempi di risposta precedenti, oltre che sulle connessioni precedenti non riuscite e con esito positivo che vengono eseguite dal dispatcher. Per ogni tentativo, la valutazione relativa alla categoria dell&#39;URI richiesto viene aggiornata.

>[!NOTE]
>
>Se non si utilizza il bilanciamento del carico, è possibile omettere questa sezione.

### Definizione delle categorie statistiche {#defining-statistics-categories}

Definite una categoria per ogni tipo di documento per il quale desiderate conservare le statistiche per la selezione del rendering. La sezione /statistics contiene una sezione /categories. Per definire una categoria, aggiungi una riga sotto la sezione /categories che include il formato seguente:

`/name { /glob "pattern"}`

La categoria `name` deve essere univoca per la fattoria. come descritto `pattern` nella sezione [Progettazione di pattern per la](#designing-patterns-for-glob-properties) sezione Proprietà della glob.

Per determinare la categoria di un URI, il dispatcher confronta l&#39;URI con ciascun pattern di categoria fino a quando non viene trovata una corrispondenza. Il dispatcher inizia con la prima categoria nell&#39;elenco e le aree in ordine. Occorre quindi inserire prima categorie con pattern più specifici.

Ad esempio, Dispatcher il dispatcher predefinito. Qualsiasi file definisce una categoria HTML e un&#39;altra categoria. La categoria HTML è più specifica e per prima cosa viene visualizzata:

```xml
/statistics
  {
  /categories
    {
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

L&#39;esempio seguente include anche una categoria per le pagine di ricerca:

```xml
/statistics
  {
  /categories
    {
      /search { /glob "*search.html" }
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

### Reflecting Server Unavailability in Dispatcher Statistics {#reflecting-server-unavailability-in-dispatcher-statistics}

La `/unavailablePenalty` proprietà imposta l&#39;ora (in secondi di un secondo) applicata alle statistiche di rendering quando una connessione al rendering non riesce. Il dispatcher aggiunge l&#39;ora alla categoria statistiche che corrisponde all&#39;URI richiesto.

Ad esempio, la sanzione viene applicata quando la connessione TCP/IP al nome host o alla porta designata non può essere stabilita, poiché AEM non è in esecuzione (e non è ascolto) o a causa di un problema di rete.

La `/unavailablePenalty` proprietà è un elemento secondario della `/farm` sezione (un elemento di pari livello della `/statistics` sezione).

Se non `/unavailablePenalty` esiste una proprietà, viene utilizzato il valore &quot;1&quot;.

```xml
/unavailablePenalty "1"
```

## Identificazione di una cartella di connessione fisso: /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

La `/stickyConnectionsFor` proprietà definisce una cartella che contiene documenti fissi; l&#39;URL viene eseguito tramite l&#39;URL. Dispatcher invia tutte le richieste, da un singolo utente, che si trovano in questa cartella alla stessa istanza di rendering. Le connessioni fissi garantiscono che i dati delle sessioni siano presenti e coerenti per tutti i documenti. Questo meccanismo utilizza il `renderid` cookie.

L&#39;esempio seguente definisce una connessione fisso alla cartella /products:

```xml
/stickyConnectionsFor "/products"
```

Quando una pagina è composta di contenuto da più nodi di contenuto, includete la `/paths` proprietà in cui sono elencati i percorsi del contenuto. Ad esempio, una pagina contiene contenuto da `/content/image`, `/content/video`e `/var/files/pdfs`. La seguente configurazione abilita le connessioni fissi per tutto il contenuto della pagina:

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### Httponly {#httponly}

Quando sono abilitate le connessioni fisse, il modulo di dispatcher imposta il `renderid` cookie. Questo cookie non ha `httponly` il flag, che dovrebbe essere aggiunto per migliorare la sicurezza. È possibile eseguire questa operazione impostando la `httpOnly` proprietà nel `/stickyConnections` nodo di un `dispatcher.any` file di configurazione. Il valore della proprietà (0 o 1) definisce se il `renderid` cookie è stato aggiunto all&#39; `HttpOnly` attributo. Il valore predefinito è 0, che significa che l&#39;attributo non verrà aggiunto.

Per ulteriori informazioni sul `httponly` flag, leggete [questa pagina](https://www.owasp.org/index.php/HttpOnly).

### secure {#secure}

Quando sono abilitate le connessioni fisse, il modulo di dispatcher imposta il `renderid` cookie. Questo cookie non ha il flag **protetto** , che dovrebbe essere aggiunto per migliorare la sicurezza. È possibile eseguire questa operazione impostando la `secure` proprietà nel `/stickyConnections` nodo di un `dispatcher.any` file di configurazione. Il valore della proprietà (0 o 1) definisce se il `renderid` cookie è stato aggiunto all&#39; `secure` attributo. Il valore predefinito è 0, che significa che l&#39;attributo verrà aggiunto se * * la richiesta in arrivo è sicura. Se il valore è impostato su 1, il flag protetto verrà aggiunto a prescindere dal fatto che la richiesta in entrata sia sicura o meno.

## Gestione degli errori di connessione rendering {#handling-render-connection-errors}

Configura il comportamento del dispatcher quando il server di rendering restituisce un errore 500 o non è disponibile.

### Specifica di una pagina di controllo dello stato {#specifying-a-health-check-page}

Utilizzate la `/health_check` proprietà per specificare un URL controllato quando si verifica un codice di stato 500. Se questa pagina restituisce anche un codice di stato 500, l&#39;istanza viene considerata non disponibile e viene applicata una pena di tempo configurabile ( `/unavailablePenalty`) al rendering prima di riprovare.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifica del ritardo di aggiornamento pagina {#specifying-the-page-retry-delay}

La proprietà/ `retryDelay` imposta l&#39;ora (in secondi) che il dispatcher attende tra round of connection tentativi con il rendering della fattoria. Per ogni round, il numero massimo di volte in cui il dispatcher cerca una connessione a un rendering è il numero di rendering nella fattoria.

Il dispatcher utilizza un valore `"1"` se `/retryDelay` non viene definito in modo esplicito. Nella maggior parte dei casi, il valore predefinito è appropriato.

```xml
/retryDelay "1"
```

### Configurazione del numero di tentativi {#configuring-the-number-of-retries}

La `/numberOfRetries` proprietà imposta il numero massimo di tentativi di connessione che il dispatcher esegue insieme al rendering. Se il dispatcher non riesce a connettersi a un rendering dopo questo numero di tentativi, il dispatcher restituisce una risposta non riuscita.

Per ogni round, il numero massimo di volte in cui il dispatcher cerca una connessione a un rendering è il numero di rendering nella fattoria. Pertanto, il numero massimo di volte in cui il dispatcher cerca una connessione è ( `/numberOfRetries`) x (il numero di rendering).

Se il valore non viene definito in modo esplicito, il valore predefinito è **5**.

```xml
/numberOfRetries "5"
```

### Utilizzo del meccanismo di failover {#using-the-failover-mechanism}

Abilitate il meccanismo di failover sulla fattoria del dispatcher per inviare nuovamente le richieste a diversi rendering quando la richiesta originale non riesce. Quando il failover è abilitato, il dispatcher ha il comportamento seguente:

* Quando una richiesta a un rendering restituisce lo stato HTTP 503 (NON DISPONIBILE), Dispatcher invia la richiesta a un altro rendering.
* Quando una richiesta a un rendering restituisce lo stato HTTP 50 x (diverso da 503), Dispatcher invia una richiesta per la pagina configurata per la `health_check` proprietà.

   * Se il controllo dello stato restituisce 500 (INTERNAL_ SERVER_ ERROR), Dispatcher invia la richiesta originale a un rendering diverso.
   * Se il controllo correttivo restituisce lo stato HTTP 200, il dispatcher restituisce l&#39;errore HTTP 500 iniziale al client.

Per abilitare il failover, aggiungete la riga seguente all&#39;azienda (o al sito Web):

```xml
/failover "1" 
```

>[!NOTE]
>
>Per riprovare le richieste HTTP contenenti un corpo, Dispatcher invia un&#39;intestazione `Expect: 100-continue` di richiesta al rendering prima di eseguire lo spooling del contenuto effettivo. CQ 5.5 con CQSE è quindi in grado di rispondere immediatamente a 100 (CONTINUA) o a un codice di errore. Anche altri contenitori servlet devono supportare questa funzione.

## Ignorare gli errori di interruzione - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>In genere questa opzione non è necessaria. Solo quando vengono visualizzati i messaggi di registro seguenti:
>
>`Error while reading response: Interrupted system call`

Qualsiasi chiamata al sistema di file con orientamento file può essere sospesa `EINTR` se l&#39;oggetto della chiamata del sistema si trova su un sistema remoto a cui accede tramite NFS. Se queste chiamate del sistema possono essere ritirate o interrotte si basano su come il file system sottostante è stato montato sul computer locale.

Utilizzate il parametro /ignoreEINTR se l&#39;istanza ha una configurazione simile e il registro contiene il seguente messaggio:

`Error while reading response: Interrupted system call`

Internamente, il dispatcher legge la risposta dal server remoto (ovvero AEM) utilizzando un ciclo che può essere rappresentato come:

`while (response not finished) {  
read more data  
}`

Questi messaggi possono essere generati quando si `EINTR` verifica nella sezione « `read more data`» e sono causati dalla ricezione di un segnale prima della ricezione di eventuali dati.

Per ignorare tali interruzioni potete aggiungere il seguente parametro a `dispatcher.any` (prima `/farms`):

`/ignoreEINTR "1"`

Impostazione `/ignoreEINTR` per `"1"` consentire a Dispatcher di continuare a leggere i dati fino a quando la risposta completa non viene letta. Il valore predefinito è 0 e disattiva l&#39;opzione.

## Progettazione di pattern per proprietà glob {#designing-patterns-for-glob-properties}

Diverse sezioni del file di configurazione del dispatcher utilizzano `glob` proprietà come criteri di selezione per le richieste client. I valori delle proprietà della glob sono pattern che il dispatcher confronta con un aspetto della richiesta, ad esempio il percorso della risorsa richiesta, o l&#39;indirizzo IP del client. Ad esempio, gli elementi della `/filter` sezione utilizzano pattern di glob per identificare i percorsi delle pagine su cui il dispatcher agisce o rifiuta.

I valori di glob possono includere caratteri jolly e caratteri alfanumerici per definire il pattern.

| Carattere jolly | Descrizione | Esempi |
|--- |--- |--- |
| `*` | Corrisponde a zero o più istanze contigue di qualsiasi carattere della stringa. Il carattere finale della corrispondenza è determinato dalle situazioni seguenti: <br/>Un carattere nella stringa corrisponde al carattere successivo del pattern e il carattere pattern ha le caratteristiche seguenti:<br/><ul><li>Non a *</li><li>Not a?</li><li>Un carattere letterale (incluso uno spazio) o una classe di caratteri.</li><li>Viene raggiunta la fine del pattern.</li></ul>All&#39;interno di una classe di caratteri, il carattere viene interpretato letteralmente. | `*/geo*` Corrisponde a qualsiasi pagina sotto il `/content/geometrixx` nodo e il `/content/geometrixx-outdoors` nodo. Le seguenti richieste HTTP corrispondono al pattern della glob: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*`<br/>Corrisponde a qualsiasi pagina sotto il `/content/geometrixx-outdoors` nodo. Ad esempio, la seguente richiesta HTTP corrisponde al pattern della glob: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Corrisponde a qualsiasi singolo carattere. Utilizzare classi di caratteri esterne. All&#39;interno di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*outdoors/??/*`<br/> Corrisponde alle pagine di qualsiasi lingua nel sito geometrixx-outdoors. Ad esempio, la seguente richiesta HTTP corrisponde al pattern della glob: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>La seguente richiesta non corrisponde al pattern della glob: <br/><ul><li>&quot; GET /content/geometrixx-outdoors/en.html &quot;</li></ul> |
| `[ and ]` | Indica l&#39;inizio e la fine di una classe di caratteri. Le classi di caratteri possono includere uno o più intervalli di caratteri e singoli caratteri.<br/>Una corrispondenza si verifica se il carattere di destinazione corrisponde a qualsiasi carattere nella classe di caratteri o all&#39;interno di un intervallo definito.<br/>Se la parentesi quadre di chiusura non è inclusa, il pattern non genera corrispondenze. | `*[o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*`<br/>Corrisponde alle seguenti richieste HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Denota un intervallo di caratteri. Da utilizzare nelle classi di caratteri. Esterno a una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[m-p]men.html*` Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Rifiuta la classe di caratteri o di caratteri che segue. Utilizzare solo per negare caratteri e intervalli di caratteri all&#39;interno delle classi di caratteri. Equivalente all &#39; `^ wildcard`. <br/>Esterno a una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[!o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` o `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Rifiuta l&#39;intervallo di caratteri o caratteri che segue. Utilizzare per negare solo caratteri e intervalli di caratteri all&#39;interno delle classi di caratteri. Equivalente al `!` carattere jolly. <br/>Esterno a una classe di caratteri, questo carattere viene interpretato letteralmente. | Sono applicati gli esempi relativi al `!` carattere jolly, sostituendo `!` i caratteri nei pattern di esempio con `^` caratteri. |


<!--- need to troubleshoot table

The following table describes the wildcard characters.

<table border="1" cellpadding="1" cellspacing="0" width="100%"> 
 <tbody> 
  <tr> 
   <th>Wildcard character</th> 
   <th>Description</th> 
   <th>Examples</th> 
  </tr> 
  <tr> 
   <td>*</td> 
   <td><p>Matches zero or more contiguous instances of any character in the string. The final character of the match is determined by either of the following situations:</p> 
    <ul> 
     <li>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics: 
      <ul> 
       <li>Not a *</li> 
       <li>Not a ?</li> 
       <li>A literal character (including a space) or a character class.</li> 
      </ul> </li> 
     <li>The end of the pattern is reached.</li> 
    </ul> <p>Inside a character class, the character is interpreted literally.</p> </td> 
   <td><p>*/geo*</p> <p>Matches any page below the /content/geometrixx node and the /content/geometrixx-outdoors node. The following HTTP requests match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx/en.html"</li> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> <p>*outdoors/*</p> <p>Matches any page below the /content/geometrixx-outdoors node. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html"</li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>?</td> 
   <td><p>Matches any single character. Use outside character classes.</p> <p>Inside a character class, this character is interpreted literally.</p> </td> 
   <td><p>*outdoors/??/*</p> <p>Matches the pages for any language in the geometrixx-outdoors site. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>The following request does not match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>[ and ]</td> 
   <td><p>Demarks the beginning and end of a character class.</p> <p>Character classes can include one or more character ranges and single characters.</p> <p>A match occurs if the target character matches any of the characters in the character class, or within a defined range.</p> <p>If the closing bracket is not included, the pattern produces no matches.</p> <p></p> <p></p> <p></p> </td> 
   <td><p>*[o]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p>*[o/]men.html*</p> <p>Matches the following HTTP requests: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
     <li> "GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>-</td> 
   <td><p>Denotes a range of characters. For use in character classes.<br /> </p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[m-p]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>!</td> 
   <td><p>Negates the character or character class that follows. Use only for negating characters and character ranges inside character classes. Equivalent to the ^ wildcard.</p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[!o]men.html*</p> <p>Matches the following HTTP request: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>Does not match the following HTTP request</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
    </ul> <p>*[!o!/]men.html*</p> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" or "GET /content/geometrixx-outdoors/en/men. html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>^</td> 
   <td><p>Negates the character or character range that follows. Use for negating only characters and character ranges inside character classes. Equivalent to the ! wildcard character.</p> <p>Outside of a character class, this charcter is interpreted literally.</p> </td> 
   <td>The examples for the ! wildcard character apply, substituting the ! characters in the example patterns with ^ characters.</td> 
  </tr> 
 </tbody> 
</table>
-->

## Registrazione {#logging}

Nella configurazione del server Web potete impostare:

* Posizione del file di registro del dispatcher.
* Il livello di registro.

Per ulteriori informazioni, consultate la documentazione del server Web e il file leggimi dell&#39;istanza di Dispatcher.

**Registri Apache ruotati/piè di pagina**

Se utilizzate un **server** Web Apache, potete usare la funzionalità standard per i registri ruotati e/o piè di pagina. Ad esempio, utilizzando i registri piè di pagina:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Questa operazione ruota automaticamente:

* il file di registro del dispatcher; con marca temporale nell&#39;estensione (logs/dispatcher. log % Y % m % d).
* settimanale (60 x 60 x 24 x 7 = 604800 secondi).

Consultate la documentazione del server Web Apache su Rotazione registro e Registri piè di pagina; ad esempio [Apache 2.2](https://httpd.apache.org/docs/2.2/logs.html).

>[!NOTE]
>
>All&#39;installazione il livello di registro predefinito è elevato (ad es. livello 3 = Debug), in modo che il dispatcher registri tutti gli errori e gli avvisi. Questo è molto utile nelle fasi iniziali.
>
>Tuttavia, questo richiede risorse aggiuntive, quindi se il dispatcher funziona senza problemi *in base ai requisiti*, è possibile (dovrebbe) abbassare il livello di registro.

### Registrazione registrazione {#trace-logging}

Tra gli altri miglioramenti apportati al dispatcher, la versione 4.2.0 introduce anche Trace Logging.

Si tratta di un livello superiore rispetto alla registrazione di debug, con informazioni aggiuntive nei registri. Aggiunge l&#39;accesso per:

* I valori delle intestazioni inoltrate;
* La regola che viene applicata a una determinata azione.

Potete abilitare la funzione di registrazione traccia impostando il livello di registro sul `4` server Web.

Di seguito è riportato un esempio di registri con il ricalco abilitato:

```xml
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Host] = "localhost:8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[User-Agent] = "curl/7.43.0"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Accept] = "*/*"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Client-Cert] = "(null)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Via] = "1.1 localhost:8443 (dispatcher)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-For] = "::1"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL] = "on"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Cipher] = "DHE-RSA-AES256-SHA"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Session-ID] = "ba931f5e4925c2dde572d766fdd436375e15a0fd24577b91f4a4d51232a934ae"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-Port] = "8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Server-Agent] = "Communique-Dispatcher"
```

E un evento registrato quando viene richiesto un file che corrisponde a una regola di blocco:

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## Conferma dell&#39;operazione di base {#confirming-basic-operation}

Per confermare operazioni di base e interazione con il server Web, il dispatcher e l&#39;istanza di AEM potete utilizzare i seguenti passaggi:

1. Impostare il `loglevel` valore su `3`.

1. Avviare il server Web; viene avviato anche il dispatcher.
1. Avviate l&#39;istanza AEM.
1. Controllate i file di registro e di errore per il server Web e il dispatcher.\
   A seconda del server Web, dovresti visualizzare messaggi quali:\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   e:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Surf il sito Web tramite il server Web. Confermate che il contenuto sia visualizzato come richiesto.\
   Ad esempio, su un&#39;installazione locale in cui AEM viene eseguito sulla porta `4502` e sul server Web all&#39; `80` accesso alla console Siti Web tramite:\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`I risultati devono essere identici. Confermate l&#39;accesso ad altre pagine con lo stesso meccanismo.

1. Verificate che la directory della cache sia in fase di compilazione.
1. Attivate una pagina per verificare che la cache sia stata scaricata correttamente.
1. Se funziona correttamente, puoi ridurre il `loglevel` valore `0`.

## Utilizzo di più dispatcher {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più Dispatcher. Ad esempio, è possibile utilizzare:

* un dispatcher per pubblicare un sito Web nella rete Intranet
* un secondo Dispatcher, con un indirizzo diverso e con impostazioni di sicurezza diverse, per pubblicare lo stesso contenuto su Internet.

In tal caso, accertatevi che ogni richiesta sia attraversata da un solo dispatcher. Un dispatcher non gestisce le richieste derivanti da un altro dispatcher. Assicuratevi quindi che entrambi i dispatcher accedano direttamente al sito Web AEM.

## Debugging {#debugging}

Quando si aggiunge l&#39;intestazione `X-Dispatcher-Info` a una richiesta, il dispatcher risponde a se la destinazione è stata memorizzata nella cache, restituita dalla cache o non memorizzata nella cache. L&#39;intestazione della risposta `X-Cache-Info` contiene queste informazioni in un modulo leggibile. Potete utilizzare queste intestazioni di risposta per risolvere problemi relativi alle risposte memorizzate nella cache dal dispatcher.

Questa funzionalità non è attivata per impostazione predefinita, pertanto, per includere l&#39;intestazione `X-Cache-Info` della risposta, l&#39;azienda deve contenere la voce seguente:

```xml
/info "1"
```

Esempio,

```xml
/farm
{
    /mywebsite
    {
        # Include X-Cache-Info response header if X-Dispatcher-Info is in request header
        /info "1"
    }
}
```

Inoltre, l&#39; `X-Dispatcher-Info` intestazione non necessita di un valore, ma se utilizzate `curl` per test dovete fornire un valore per inviare l&#39;intestazione, ad esempio:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/we-retail/us/en.html
```

Di seguito è riportato un elenco contenente le intestazioni di risposta `X-Dispatcher-Info` che restituiranno:

* **nella cache**\
   Il file di destinazione è contenuto nella cache e il dispatcher ha stabilito che è valido distribuirlo.
* **caching**\
   Il file di destinazione non è contenuto nella cache e il dispatcher ha stabilito che è valido memorizzare nella cache l&#39;output e distribuirlo.
* **caching: stat file è più recente**
Il file di destinazione è contenuto nella cache, ma viene invalidato da un file stat più recente. Il dispatcher eliminerà il file di destinazione, lo ricreerà dall&#39;output e lo consegna.
* **non cachable: no document root**
The farm&#39;s configuration doesn&#39;t contain a document root (configuration element `cache.docroot`).
* **non cachable: percorso file cache troppo lungo**\
   Il file di destinazione, ovvero la concatenazione di root e file URL, supera il nome file più lungo possibile sul sistema.
* **non cachable: percorso file temporaneo troppo lungo**\
   Il modello nome file temporaneo supera il nome file più lungo possibile sul sistema. Prima di creare o sovrascrivere il file memorizzato nella cache, il dispatcher crea innanzitutto un file temporaneo. Il nome file temporaneo è il nome del file di destinazione con i caratteri `_YYYYXXXXXX` aggiunti, dove viene `Y` sostituito e `X` verrà sostituito per creare un nome univoco.
* **non cachable: l&#39;URL della richiesta non ha estensione**\
   L&#39;URL della richiesta non ha estensione, o esiste un percorso che segue l&#39;estensione del file, ad esempio: `/test.html/a/path`.
* **non cachable: la richiesta non è GET o HEAD**
Il metodo HTTP non è né GET né HEAD. Il dispatcher presuppone che l&#39;output contenga dati dinamici che non dovrebbero essere memorizzati nella cache.
* **non cachable: request contains a query string**\
   La richiesta conteneva una stringa di query. Il dispatcher presuppone che l&#39;output dipenda dalla stringa query specificata e quindi non la cache.
* **non cachable: session manager did&#39;t authentof**\
   La cache dell&#39;azienda è governata da un manager sessione (la configurazione contiene un `sessionmanagement` nodo) e la richiesta non contiene le informazioni di autenticazione appropriate.
* **non cachable: la richiesta contiene l&#39;autorizzazione**\
   All&#39;azienda non è consentito memorizzare output ( `allowAuthorized 0`) e la richiesta contiene informazioni sull&#39;autenticazione.
* **non cachable: target è una directory**\
   Il file di destinazione è una directory. Questo potrebbe puntare ad alcuni errori concettuali, in cui un URL e alcuni URL secondari contengono entrambi output cachable, ad esempio se una richiesta al `/test.html/a/file.ext` primo e contiene output cachable, il dispatcher non sarà in grado di memorizzare nella cache l&#39;output di una richiesta successiva a `/test.html`.
* **non cachable: l&#39;URL della richiesta ha una barra finale**\
   L&#39;URL della richiesta ha una barra finale.
* **non cachable: URL della richiesta non nelle regole della cache**\
   Le regole della cache della fattoria rifiutano esplicitamente la memorizzazione nella cache dell&#39;output di alcuni URL della richiesta.
* **non cachable: controllo delle autorizzazioni rifiutato**\
   Il controllo dell&#39;autorizzazione della fattoria ha rifiutato l&#39;accesso al file memorizzato nella cache.
* **non cachable: session not valid**
the farm&#39;s cache is governed by a session manager (configuration contains a `sessionmanagement` node) and the user&#39;s session&#39;s session is not or no no.
* **non cachable: response Contiene`no_cache `** il server remoto restituito da un `Dispatcher: no_cache` &#39;intestazione, impedendo al dispatcher di memorizzare nella cache l&#39;output.
* **non cachable: response content length is zero**
La lunghezza del contenuto della risposta è zero; il dispatcher non crea un file con lunghezza zero.
