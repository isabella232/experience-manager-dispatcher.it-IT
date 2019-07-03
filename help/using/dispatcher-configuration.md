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
source-git-commit: a997d2296e80d182232677af06a2f4ab5a14bfd5

---


# Configuring Dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporata nella documentazione per una versione precedente di AEM.

Nelle sezioni seguenti viene descritto come configurare vari aspetti del dispatcher.

## Support for IPv4 and IPv6 {#support-for-ipv-and-ipv}

Tutti gli elementi di AEM e Dispatcher possono essere installati sia nelle reti ipv 4 che ipv 6. See [IPV4 and IPV6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes).

## Dispatcher Configuration Files {#dispatcher-configuration-files}

By default the Dispatcher configuration is stored in the `dispatcher.any` text file, though you can change the name and location of this file during installation.

Il file di configurazione contiene una serie di proprietà con valori singolo o con più valutazioni che controllano il funzionamento del dispatcher:

* Property names are prefixed with a forward slash `/`.
* Multi-valued properties enclose child items using braces `{ }`.

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

For example, if the files `farm_1.any` through to `farm_5.any` contain the configuration of farms one to five, you can include them as follows:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Using Environment Variables {#using-environment-variables}

È possibile utilizzare variabili d&#39;ambiente in proprietà con valore stringa nel dispatcher. qualsiasi file, anziché codificare i valori. To include the value of an environment variable, use the format `${variable_name}`.

For example, if the dispatcher.any file is located in the same directory as the cache directory, the following value for the [docroot](dispatcher-configuration.md#main-pars-title-30) property can be used:

```xml
/docroot "${PWD}/cache"
```

As another example, if you create an environment variable named `PUBLISH_IP` that stores the hostname of the AEM publish instance, the following configuration of the [/renders](dispatcher-configuration.md#main-pars-127-25-0008) property can be used:

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Naming the Dispatcher Instance {#naming-the-dispatcher-instance-name}

Use the `/name` property to specify a unique name to identify your Dispatcher instance. The `/name` property is a top-level property in the configuration structure.

## Defining Farms {#defining-farms-farms}

The `/farms` property defines one or more sets of Dispatcher behaviors, where each set is associated with different web sites or URLs. The `/farms` property can include a single farm or multiple farms:

* Utilizzate una sola fattoria per gestire tutte le pagine Web e i siti Web allo stesso modo.
* Create più aziende quando diverse aree del sito Web o siti Web diversi richiedono un comportamento di Dispatcher diverso.

The `/farms` property is a top-level property in the configuration structure. To define a farm, add a child property to the `/farms` property. Utilizzate un nome di proprietà che identifica in modo univoco la fattoria nell&#39;istanza di Dispatcher.

The `/*farmname*` property is multi-valued, and contains other properties that define Dispatcher behavior:

* Gli URL delle pagine a cui si applica la fattoria.
* Uno o più URL del servizio (in genere delle istanze di pubblicazione AEM) da utilizzare per il rendering dei documenti.
* Statistiche da utilizzare per il bilanciamento del carico più renderer di documenti.
* Diversi altri comportamenti, quali i file alla cache e dove.

Il valore può contenere qualsiasi carattere alfanumerico (a-z, 0-9). The following example shows the skeleton definition for two farms named `/daycom` and `/docsdaycom`:

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
>Se utilizzate più di una farm farm, l&#39;elenco viene valutato in basso. This is particularly relevant when defining [Virtual Hosts](dispatcher-configuration.md#main-pars-117-15-0006) for your websites.

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

## Specify a Default Page (IIS Only) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>The `/homepage`parameter (IIS only) no longer works. Instead, you should use the [IIS URL Rewrite Module](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>If you are using Apache, you should use the `mod_rewrite` module. See the Apache web site documentation for information about `mod_rewrite` (for example, [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). When using `mod_rewrite`, it is advisable to use the flag ** [&#39;passthrough|PT&#39; (pass through to next handler)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** to force the rewrite engine to set the `uri` field of the internal `request_rec` structure to the value of the `filename` field.

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

## Specifying the HTTP Headers to Pass Through {#specifying-the-http-headers-to-pass-through-clientheaders}

The `/clientheaders` property defines a list of HTTP headers that Dispatcher passes from the client HTTP request to the renderer (AEM instance).

Per impostazione predefinita, il dispatcher ha inoltrato le intestazioni HTTP standard all&#39;istanza AEM. In alcuni casi, è possibile che vengano inoltrate intestazioni aggiuntive o che vengano rimosse intestazioni specifiche:

* Aggiungete intestazioni, come intestazioni personalizzate, che l&#39;istanza AEM prevede nella richiesta HTTP.
* Rimuovere intestazioni, quali intestazioni di autenticazione, pertinenti solo al server Web.

Se personalizzate il set di intestazioni da superare, dovete specificare un elenco completo delle intestazioni, compresi quelli normalmente inclusi per impostazione predefinita.

For example, a Dispatcher instance that handles page activation requests for publish instances requires the `PATH` header in the `/clientheaders` section. The `PATH` header enables communication between the replication agent and the dispatcher.

The following code is an example configuration for `/clientheaders`:

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

## Identifying Virtual Hosts {#identifying-virtual-hosts-virtualhosts}

The `/virtualhosts` property defines a list of all hostname/URI combinations that Dispatcher accepts for this farm. Potete usare il carattere asterisco (&quot; *&quot;) come carattere jolly. Values for the / `virtualhosts` property use the following format:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Facoltativo) `https://` O `https://.`
* `host`: Il nome o l&#39;indirizzo IP del computer host e il numero di porta, se necessario. (See [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
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

The following configuration handles *all* requests:

```xml
   /virtualhosts
    {
    "*"
    }
```

### Resolving the Virtual Host {#resolving-the-virtual-host}

When Dispatcher receives an HTTP or HTTPS request, it finds the virtual host value that best-matches the `host,` `uri`, and `scheme` headers of the request. Dispatcher evaluates the values in the `virtualhosts` properties in the following order:

* Il dispatcher inizia dalla fattoria più bassa e avanza verso l&#39;alto nel dispatcher. any file.
* For each farm, Dispatcher begins with the topmost value in the `virtualhosts` property and progresses down the list of values.

Il dispatcher trova il valore host virtuale che corrisponde meglio nel modo seguente:

* The first-encountered virtual host that matches all three of the `host`, the `scheme`, and the `uri` of the request is used.
* If no `virtualhosts` values has `scheme` and `uri` parts that both match the `scheme` and `uri` of the request, the first-encountered virtual host that matches the `host` of the request is used.
* If no `virtualhosts` values have a host part that matches the host of the request, the topmost virtual host of the topmost farm is used.

Therefore, you should place your default virtual host at the top of the `virtualhosts` property in the topmost farm of your dispatcher.any file.

### Example Virtual Host Resolution {#example-virtual-host-resolution}

The following example represents a snippet from a dispatcher.any file that defines two Dispatcher farms, and each farm defines a `virtualhosts` property.

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

## Enabling Secure Sessions - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized`**per** abilitare questa funzione deve essere impostato `"0"` nella `/cache` sezione.

Create una sessione protetta per accedere alla farm farm, in modo che gli utenti debbano accedere per accedere a qualsiasi pagina della fattoria. Dopo aver eseguito l&#39;accesso, gli utenti possono accedere alle pagine della fattoria. See [Creating a Closed User Group](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) for information about using this feature with CUGs. Also, see the Dispatcher [Security Checklist](/help/using/security-checklist.md) before going live.

The `/sessionmanagement` property is a subproperty of `/farms`.

>[!CAUTION]
>
>Se le sezioni del sito Web usano requisiti di accesso diversi, è necessario definire più aziende.

**/sessionmanagement** presenta diversi parametri secondari:

**/directory** (obbligatorio)

La directory in cui memorizzate le informazioni della sessione. Se la directory non esiste, viene creata.

>[!CAUTION]
>
> When configuring the directory sub-parameter **do not** point to the root folder (`/directory "/"`) as it can cause serious problems. Specificate sempre il percorso della cartella in cui memorizzare le informazioni della sessione. Esempio:

```xml
/sessionmanagement 
  { 
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (facoltativo)

Modalità di codifica delle informazioni della sessione. Utilizzare &quot;md 5&quot; per la cifratura utilizzando l&#39;algoritmo md 5 o &quot;hex&quot; per la codifica esadecimale. Se si cifrano i dati della sessione, un utente con accesso al file system non può leggere il contenuto della sessione. Il valore predefinito è &quot;md 5&quot;.

**/header** (facoltativo)

Nome dell&#39;intestazione o del cookie HTTP che memorizza le informazioni sull&#39;autorizzazione. If you store the information in the http header, use `HTTP:<*header-name*>`. To store the information in a cookie, use `Cookie:<header-name>`. If you do not specify a value `HTTP:authorization` is used.

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

## Defining Page Renderers {#defining-page-renderers-renders}

La proprietà /renders definisce l&#39;URL a cui Dispatcher invia le richieste per il rendering di un documento. The following example `/renders` section identifies a single AEM instance for rendering:

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

### Renders options {#renders-options}

**/timeout**

Specifica il timeout della connessione che accede all&#39;istanza AEM in millisecondi. Il valore predefinito è &quot;0&quot;, comportando l&#39;attesa a tempo indefinito del dispatcher.

**/receiveTimeout**

Specifica il tempo in millisecondi consentito per la risposta. Il valore predefinito è &quot;600000&quot;, che causava l&#39;attesa di 10 minuti da parte del dispatcher. Un&#39;impostazione di &quot;0&quot; elimina completamente il timeout.\
Se viene raggiunto il timeout durante l&#39;analisi delle intestazioni di risposta, viene restituito uno stato HTTP di 504 (Gateway errato). Se il timeout viene raggiunto durante la lettura del corpo della risposta, il dispatcher restituirà la risposta incompleta al client, ma elimina eventuali file della cache che potrebbero essere stati scritti.

**/ipv4**

Specifies whether Dispatcher uses the `getaddrinfo` function (for IPv6) or the `gethostbyname` function (for IPv4) for obtaining the IP address of the render. A value of 0 causes `getaddrinfo` to be used. A value of 1 causes `gethostbyname` to be used. Il valore predefinito è 0.

La funzione getaddrinfo restituisce un elenco di indirizzi IP. Il dispatcher ripeterà l&#39;elenco degli indirizzi finché non stabilisce una connessione TCP/IP. Pertanto, la proprietà ipv 4 è importante quando il nome host di rendering è associato a indirizzi IP con più stati e l&#39;ospitante, in risposta alla funzione getaddrinfo, restituisce un elenco di indirizzi IP sempre nello stesso ordine. In questa situazione, è necessario utilizzare la funzione gethostbyname in modo che l&#39;indirizzo IP con cui si collega il dispatcher sia casuale.

Il bilanciamento del carico di Amazon Elastic (ELB) è un servizio tale che risponde a getaddrinfo con un elenco possibile di indirizzi IP ordinato.

**/secure**

If the `/secure` property has a value of &quot;1&quot; Dispatcher uses HTTPS to communicate with the AEM instance. For additional details, also see [Configuring Dispatcher to Use SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

With Dispatcher version **4.1.6**, you can configure the `/always-resolve` property as follows:

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

## Configuring Access to Content {#configuring-access-to-content-filter}

Use the `/filter` section to specify the HTTP requests that Dispatcher accepts. Tutte le altre richieste vengono inviate al server Web con un codice di errore 404 (pagina non trovata). If no `/filter` section exists, all requests are accepted.

**Note:** Requests for the [statfile](dispatcher-configuration.md#main-pars-title-28) are always rejected.

>[!CAUTION]
>
>See the [Dispatcher Security Checklist](security-checklist.md) for further considerations when restricting access using Dispatcher. Also, read the [AEM Security Cheklist](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) for additional security details regarding your AEM installation.

La sezione /filter consiste in una serie di regole che possono negare o consentire l&#39;accesso al contenuto in base ai pattern nella parte richiesta della richiesta HTTP. Utilizzate una strategia meteortelist per la sezione /filter:

* Innanzitutto, nega l&#39;accesso a tutti gli elementi.
* Consente l&#39;accesso al contenuto in base alle esigenze.

### Defining a Filter {#defining-a-filter}

Each item in the `/filter` section includes a type and a pattern that is matched with a specific element of the request line or the entire request line. Ciascun filtro può contenere i seguenti elementi:

* **Tipo**: Indica `/type` se consentire o negare l&#39;accesso per le richieste che corrispondono al pattern. The value can be either `allow` or `deny`.

* **Elemento della riga richiesta:** Includete `/method``/url`, `/query`o `/protocol` e un pattern per le richieste di filtraggio in base a queste parti specifiche della porzione di richiesta della richiesta HTTP. Il filtro sugli elementi della riga di richiesta (anziché sull&#39;intera riga di richiesta) è il metodo di filtro preferito.

* **Elementi avanzati della riga richiesta:** A partire da Dispatcher 4.2.0, sono disponibili quattro nuovi elementi filtro. These new elements are `/path`, `/selectors`, `/extension` and `/suffix` respectively. Includete uno o più elementi per controllare ulteriormente i pattern URL.

>[!NOTE]
>
>For more information about what part of the request line each of these elements references, see the [Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki page.

* **proprietà glob**: La `/glob` proprietà viene utilizzata per corrispondere all&#39;intera riga di richiesta della richiesta HTTP.

>[!CAUTION]
>
>Il filtraggio con i globi è obsoleto nel dispatcher. As such, you should avoid using globs in the `/filter` sections since it may lead to security issues. Di conseguenza, invece di:

`/glob "* *.css *"`

dovreste usare

`/url "*.css"`

#### The request-line Part of HTTP Requests {#the-request-line-part-of-http-requests}

HTTP/1.1 defines the [request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) as follows:

*Request Request-URI HTTP-Version*&lt; CRLF &gt;

I caratteri &lt; CRLF &gt; riportavano un ritorno a capo seguito da un feed di riga. L&#39;esempio seguente è la riga di richiesta che viene recisa quando un client richiede la pagina en del sito Geometrixx-Outoors:

GET /content/geometrixx-outdoors/en.html HTTP .1 .1 &lt; CRLF &gt;

I pattern devono prendere in considerazione gli spazi nella riga di richiesta e i caratteri &lt; CRLF &gt;.

#### Double-quotes vs Single-quotes {#double-quotes-vs-single-quotes}

When creating your filter rules, use double quotation marks `"pattern"` for simple patterns. If you are using Dispatcher 4.2.0 or later and your pattern includes a regular expression, you must enclose the regex pattern `'(pattern1|pattern2)'` within single quotation marks.

#### Regular Expressions {#regular-expressions}

Dopo il dispatcher 4.2.0, potete includere espressioni regolari estese POSIX nei pattern di filtro.

#### Troubleshooting Filters {#troubleshooting-filters}

If your filters are not triggering in the way you would expect, enable [Trace Logging](#trace-logging) on dispatcher so you can see which filter is intercepting the request.

#### Example Filter: Deny All {#example-filter-deny-all}

La sezione di filtro seguente fa sì che il dispatcher neghi le richieste di tutti i file. È consigliabile negare l&#39;accesso a tutti i file e consentire l&#39;accesso a specifiche aree.

```xml
  /0001  { /glob "*" /type "deny" }
```

Le richieste inviate a un&#39;area rifiutata in modo esplicito producono un codice di errore 404 (pagina non trovata) restituita.

#### Example Filter: Deny Acess to Specific Areas {#example-filter-deny-acess-to-specific-areas}

I filtri consentono inoltre di negare l&#39;accesso a vari elementi, ad esempio pagine ASP e aree sensibili all&#39;interno di un&#39;istanza di pubblicazione. Il filtro seguente nega l&#39;accesso alle pagine ASP:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Example Filter: Enable POST Requests {#example-filter-enable-post-requests}

Il filtro di esempio seguente consente di inviare i dati del modulo tramite il metodo POST:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Example Filter: Allow Access to the Workflow Console {#example-filter-allow-access-to-the-workflow-console}

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

#### Example filter: Using Regular Expressions {#example-filter-using-regular-expressions}

Questo filtro abilita le estensioni nelle directory di contenuto non pubbliche utilizzando un&#39;espressione regolare, definita qui tra virgolette singole:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Example filter: Filter Additional Elements of a Request URL {#example-filter-filter-additional-elements-of-a-request-url}

Below is a rule example that blocks content grabbing from the `/content` path and its subtree, using filters for path, selectors and extensions:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### Example /filter section {#example-filter-section}

Durante la configurazione di Dispatcher, è necessario limitare il più possibile l&#39;accesso esterno. L&#39;esempio seguente fornisce un accesso minimo ai visitatori esterni:

* `/content`
* contenuti vari quali design e librerie client; ad esempio:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

After you create filters, [test page access](dispatcher-configuration.md#main-pars-title-19) to ensure your AEM instance is secure.

The following /filter section of the dispatcher.any file can be used as a basis in your [Dispatcher configuration](dispatcher-configuration.md) file.

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
>Se utilizzato con Apache, progettate i pattern URL del filtro in base alla proprietà dispatcheruseprocessedurl del modulo Dispatcher. (See [Apache Web Server - Configure your Apache Web Server for Dispatcher](dispatcher-install.md#main-pars-55-35-1022).)

>[!NOTE]
>
>I filtri 0030 e 0031 relativi ai contenuti multimediali dinamici sono applicabili a AEM 6.0 e versioni successive.

Se scegliete di estendere l&#39;accesso, prendete in considerazione le seguenti raccomandazioni:

* External access to `/admin` should always be *completely* disabled if you are using CQ version 5.4 or an earlier version.

* Care must be taken when allowing access to files in `/libs`. L&#39;accesso deve essere consentito su base individuale.
* Rifiuta l&#39;accesso alla configurazione di replica in modo che non possa essere visualizzato:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Rifiuta l&#39;accesso al proxy inverso Google Gadgets:

   * `/libs/opensocial/proxy*`

Depending on your installation, there might be additional resources under `/libs`, `/apps` or elsewhere, that must be made available. You can use the `access.log` file as one method of determining resources that are being accessed externally.

>[!CAUTION]
>
>L&#39;accesso alle console e alle directory può presentare un rischio di protezione per gli ambienti di produzione. A meno che non abbiano giustificazione esplicita, dovrebbero rimanere disattivate (commentato).

>[!CAUTION]
>
>If you are [using reports in a publish environment](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) you should configure Dispatcher to deny access to `/etc/reports` for external visitors.

### Restricting Query Strings {#restricting-query-strings}

Since Dispatcher version 4.1.5, use the `/filter` section to restrict query strings. It is highly recommended to explicitly allow query strings and exclude generic allowance through `allow` filter elements.

A single entry can have either *glob* or some combination of *method*,*url*,*query* and *version* but not both. The following example allows the `a=*` query string and denies all other query strings for URLs that resolve to the `/etc` node:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>If a rule contains a `/query`, it will only match requests that contain a query string and match the provided query pattern.
>
>In above example, if requests to `/etc` that have no query string should be allowed as well, the following rules would be required:


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Testing Dispatcher Security {#testing-dispatcher-security}

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
* /content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)
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

## Enabling Access to Vanity URLs {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

Configurare il dispatcher per abilitare l&#39;accesso agli URL personalizzati configurati per le pagine CQ o AEM.

Quando l&#39;accesso agli URL personalizzati è abilitato, Dispatcher chiama periodicamente un servizio che viene eseguito sull&#39;istanza di rendering per ottenere un elenco di URL personalizzati. Dispatcher memorizza questo elenco in un file locale. When a request for a page is denied due to a filter in the `/filter` section, Dispatcher consults the list of vanity URLs. Se l&#39;URL rifiutato è presente nell&#39;elenco, il dispatcher consente l&#39;accesso all&#39;URL personalizzato.

To enable access to vanity URLs, add a `/vanity_urls` section to the `/farms` section, similar to the following example:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` La sezione contiene le proprietà seguenti:

* `/url`: Percorso del servizio URL personalizzato che viene eseguito nell&#39;istanza di rendering. The value of this property must be `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`: Percorso del file locale in cui Dispatcher memorizza l&#39;elenco degli URL personalizzati. Assicuratevi che il dispatcher abbia accesso in scrittura a questo file.
* `/delay`: (Secondi) L&#39;ora tra chiamate al servizio URL personalizzato.

>[!NOTE]
>
>If your render is an instance of AEM you must install the [VanityURLS-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) package to install the vanity URL service. (See [Signing In to Package Share](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare).)

Utilizzate la seguente procedura per abilitare l&#39;accesso agli URL personalizzati.

1. Se il servizio di rendering è un&#39;istanza di AEM, installate il pacchetto com. adobe. granite. dispatcher. vanityurl. content nell&#39;istanza di pubblicazione (consultate la nota sopra).
1. For each vanity URL that you have configured for an AEM or CQ page, ensure that the ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` configuration denies the URL. Se necessario, aggiungete un filtro che nega l&#39;URL.
1. Add the `/vanity_urls` section below `/farms`.
1. Riavviate Apache Web Server.

## Forwarding Syndication Requests - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

Le richieste di sindacazione sono solitamente destinate esclusivamente al dispatcher, quindi per impostazione predefinita non vengono inviate al renderer (ad esempio, un&#39;istanza di AEM).

Se necessario, impostate la proprietà /propagateSyndPost su &quot;1&quot; per inoltrare le richieste di sindacazione al dispatcher. Se impostata, dovete accertarvi che le richieste POST non vengano rifiutate nella sezione del filtro.

## Configuring the Dispatcher Cache - /cache {#configuring-the-dispatcher-cache-cache}

The `/cache` section controls how Dispatcher caches documents. Configurate diverse sottoproprietà per implementare le strategie di caching:


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
>For permission-sensitive caching, read [Caching Secured Content](permissions-cache.md).

### Specifying the Cache Directory {#specifying-the-cache-directory}

The `/docroot` property identifies the directory where cached files are stored.

>[!NOTE]
>
>Il valore deve essere lo stesso percorso della radice del documento del server Web in modo che il dispatcher e il server Web gestiscono gli stessi file.\
>Il server Web è responsabile della distribuzione del codice di stato corretto quando viene utilizzato il file della cache del dispatcher, per questo è importante che sia accessibile anche.

Se utilizzate più aziende, ogni fattoria deve utilizzare un&#39;altra radice documento.

### Naming the Statfile {#naming-the-statfile}

The `/statfile` property identifies the file to use as the statfile. Il dispatcher usa questo file per registrare l&#39;ora dell&#39;aggiornamento del contenuto più recente. Il file statico può essere qualsiasi file presente sul server Web.

Lo statfile non ha alcun contenuto. Quando il contenuto viene aggiornato, il dispatcher aggiorna la marca temporale. Il file statico predefinito è denominato. stat e viene memorizzato nel docroot. Il dispatcher blocca l&#39;accesso al statfile statico.

>[!NOTE]
>
>If `/statfileslevel` is configured, Dispatcher ignores the `/statfile` property and uses .stat as the name.

### Serving Stale Documents When Errors Occur {#serving-stale-documents-when-errors-occur}

The `/serveStaleOnError` property controls whether Dispatcher returns invalidated documents when the render server returns an error. Per impostazione predefinita, quando un statfile statico viene toccato e invalida il contenuto memorizzato nella cache, il contenuto memorizzato nella cache viene eliminato dal dispatcher alla sua successiva richiesta.

If `/serveStaleOnError` is set to &quot;1&quot;, Dispatcher does not delete invalidated content from the cache unless the render server returns a successful response. Una risposta di 5 xx da AEM o un timeout di connessione causa la trasmissione del contenuto non aggiornato e la risposta con lo stato HTTP di 111 (convalida non riuscita).

### Caching When Authentication is Used {#caching-when-authentication-is-used}

The `/allowAuthorized` property controls whether requests that contain any of the following authentication information are cached:

* The `authorization` header.
* A cookie named `authorization`.
* A cookie named `login-token`.

Per impostazione predefinita, le richieste che includono queste informazioni di autenticazione non vengono memorizzate nella cache perché l&#39;autenticazione non viene eseguita quando viene restituito un documento nella cache al client. Questa configurazione impedisce al dispatcher di distribuire i documenti memorizzati nella cache agli utenti che non dispongono dei diritti necessari.

Tuttavia, se i vostri requisiti consentono la memorizzazione nella cache dei documenti autenticati, impostate /allowAuthorized su uno:

`/allowAuthorized "1"`

>[!NOTE]
>
>To enable session management (using the `/sessionmanagement` property), the `/allowAuthorized` property must be set to `"0"`.

### Specifying the Documents to Cache {#specifying-the-documents-to-cache}

The `/rules` property controls which documents are cached according to the document path. A prescindere dalla proprietà /rules, Dispatcher non memorizza mai nella cache un documento nelle seguenti circostanze:

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
>I metodi GET o HEAD (per i metodi dell&#39;intestazione HTTP) possono essere memorizzati nella cache dal dispatcher. For additional information on response header caching, see the [Caching HTTP Response Headers](dispatcher-configuration.md#caching-http-response-headers) section.

Each item in the /rules property includes a [glob](#designing-patterns-for-glob-properties) pattern and a type:

* Il pattern di glob viene utilizzato per far corrispondere il percorso del documento.
* Il tipo indica se memorizzare nella cache i documenti che corrispondono al pattern della glob. Il valore può essere consentito (per memorizzare nella cache il documento) o negare (per eseguire il rendering sempre del documento).

Se non hai pagine dinamiche (oltre quelle già escluse dalle regole elencate), puoi configurare il dispatcher in modo da memorizzare tutti gli elementi nella cache. La sezione delle regole si presenta come segue:

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

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

Sui server Web Apache potete comprimere i documenti memorizzati nella cache. La compressione consente ad Apache di restituire il documento in un modulo compresso, se richiesto dal client. Compression is done automatically by enabling the Apache module `mod_deflate`, for example:

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

### Invalidating Files by Folder Level {#invalidating-files-by-folder-level}

Use the `/statfileslevel` property to invalidate cached files according to their path:

* Dispatcher creates `.stat`files in each folder from the docroot folder to the level that you specify. La cartella docroot è livello 0.
* Files are invalidated by touching the `.stat` file. The `.stat` file&#39;s last modification date is compared to the last modification date of a cached document. The document is re-fetched if the `.stat` file is newer.

* When a file located at a certain level is invalidated then **all** `.stat` files from the docroot **to** the level of the invalidated file or the configured `statsfilevel` (whichever is smaller) will be touched.

   * For example: if you set the `statfileslevel` property to 6 and a file is invalidated at level 5 then every `.stat` file from docroot to 5 will be touched. Continuate con questo esempio, se un file viene invalidato al livello 7 e quindi ogni. `stat` da docroot a 6 viene toccato (dal momento `/statfileslevel = "6"`).

Sono interessate solo le risorse** lungo il percorso** nel file invalidated. Consider the following example: a website uses the structure `/content/myWebsite/xx/.` If you set `statfileslevel` as 3, a `.stat`file is created as follows:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

When a file in `/content/myWebsite/xx` is invalidated then every `.stat` file from docroot down to `/content/myWebsite/xx`is touched. This would be the case only for `/content/myWebsite/xx` and not for example `/content/myWebsite/yy` or `/content/anotherWebSite`.

>[!NOTE]
>
>Invalidation can be prevented by sending an additional Header `CQ-Action-Scope:ResourceOnly`. Questo può essere utilizzato per svuotare determinate risorse senza invalidare altre parti della cache. See [this page](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) and [Manually Invalidating the Dispatcher Cache](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) for additional details.

>[!NOTE]
>
>If you specify a value for the `/statfileslevel` property, the `/statfile` property is ignored.

### Automatically Invalidating Cached Files {#automatically-invalidating-cached-files}

The `/invalidate` property defines the documents that are automatically invalidated when content is updated.

Con la convalida automatica, il dispatcher non elimina i file memorizzati nella cache dopo un aggiornamento del contenuto, ma ne verifica la validità al momento della successiva richiesta. I documenti nella cache che non vengono invalidati automaticamente rimarranno nella cache finché non vengono eliminati esplicitamente da un aggiornamento del contenuto.

La convalida automatica viene generalmente utilizzata per le pagine HTML. Le pagine HTML contengono spesso collegamenti verso altre pagine, rendendo difficile determinare se un aggiornamento del contenuto influisce su una pagina. Per verificare che tutte le pagine rilevanti vengano invalidate quando il contenuto viene aggiornato, invalida automaticamente tutte le pagine HTML. La seguente configurazione invalida tutte le pagine HTML:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

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

### Using custom invalidation scripts {#using-custom-invalidation-scripts}

La proprietà /invalidateHandler consente di definire uno script chiamato per ogni richiesta di annullamento della validità ricevuta dal dispatcher.

Viene richiamata con i seguenti argomenti:

* Punto di controllo\
   Percorso del contenuto invalidato
* Azione\
   Azione di replica (ad esempio Attiva, Disattiva)
* Ambito azione\
   The replication Action&#39;s Scope (empty, unless a header of `CQ-Action-Scope: ResourceOnly` is sent, see [Invalidating Cached Pages from AEM](page-invalidate.md) for details)

Questo può essere utilizzato per proteggere molti casi d&#39;uso diversi, ad esempio la nullità di altre caching specifiche dell&#39;applicazione, o per gestire i casi in cui l&#39;URL esternalizzato di una pagina e la sua posizione nel docroot non corrispondono al percorso del contenuto.

Di seguito viene eseguito il login di script di ogni richiesta invalidata a un file.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### sample invalidation handler script {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limiting the Clients That Can Flush the Cache {#limiting-the-clients-that-can-flush-the-cache}

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

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>Si consiglia di definire /allowedClients.
>
>In caso contrario, un client può emettere una chiamata per cancellare la cache; se questa operazione viene eseguita ripetutamente, può avere forte impatto sulle prestazioni del sito.

### Ignoring URL Parameters {#ignoring-url-parameters}

`ignoreUrlParams` La sezione definisce i parametri degli URL che vengono ignorati quando si determina se una pagina viene memorizzata nella cache o inviata dalla cache:

* Quando un URL di richiesta contiene parametri che vengono ignorati, la pagina viene memorizzata nella cache.
* Quando un URL di richiesta contiene uno o più parametri non ignorati, la pagina non viene memorizzata nella cache.

Quando un parametro viene ignorato per una pagina, la pagina viene memorizzata nella cache al primo avvio della pagina. Le richieste successive per la pagina vengono servite della pagina memorizzata nella cache, indipendentemente dal valore del parametro nella richiesta.

To specify which parameters are ignored, add glob rules to the `ignoreUrlParams` property:

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

Using the example `ignoreUrlParams` value, the following HTTP request causes the page to be cached because the `q` parameter is ignored:

```xml
GET /mypage.html?q=5
```

Using the example `ignoreUrlParams` value, the following HTTP request causes the page to **not** be cached because the `p` parameter is not ignored:

```xml
GET /mypage.html?q=5&p=4
```

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

### Caching HTTP Response Headers {#caching-http-response-headers}

>[!NOTE]
>
>This feature is avaiable with version **4.1.11** of the Dispatcher.

The `/headers` property allows you to define the HTTP header types that are going to be cached by the Dispatcher. Sulla prima richiesta a una risorsa non memorizzata nella cache, tutte le intestazioni corrispondenti a uno dei valori configurati (vedere la configurazione di seguito) sono memorizzate in un file separato, accanto al file della cache. Nelle richieste successive alla risorsa memorizzata nella cache, le intestazioni memorizzate vengono aggiunte alla risposta.

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
>Inoltre, i caratteri glogging dei file non sono consentiti. For more details, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Se devi memorizzare e distribuire intestazioni di risposta etag da AEM, effettua le seguenti operazioni:
>
>* Add the header name in the `/cache/headers`section.
>* Add the following [Apache directive](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) in the Dispatcher related section:
>



```xml
FileETag none
```

### Dispatcher Cache File Permissions {#dispatcher-cache-file-permissions}

The `mode` property specifies what file permissions are applied to new directories and files in the cache. This setting is restricted by the `umask` of the calling process. Si tratta di un numero ottale costruito partendo dalla somma di uno o più dei seguenti valori:

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

### Throttling .stat file touching {#throttling-stat-file-touching}

With the default `/invalidate` property, every activation effectively invalidates all `.html` files (when their path matches the `/invalidate` section). Su un sito Web con traffico considerevole, più, le attivazioni successive aumenteranno il carico della CPU sul backend. In such a scenario, it would be desirable to &quot;throttle&quot; `.stat` file touching to keep the website responsive. You can do this by using the `/gracePeriod` property.

The `/gracePeriod` property defines the number of seconds a stale, auto-invalidated resource may still be served from the cache after the last occuring activation. La proprietà può essere utilizzata in una configurazione in cui un batch di attivazioni altrimenti invaliderebbe l&#39;intera cache. Il valore consigliato è 2 secondi.

For additional details, also read the `/invalidate` and `/statfileslevel`sections above.

## Configuring Time Based Cache Invalidation - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

If set, the `enableTTL` property will evaluate the response headers from the backend, and if they contain a `Cache-Control` max-age or `Expires` date, an auxiliary, empty file next to the cache file is created, with modification time equal to the expiry date. Quando il file memorizzato nella cache viene richiesto oltre il tempo di modifica, viene richiesto automaticamente dal back-end.

You can enable the feature by adding this line to the `dispatcher.any` file:

```xml
/enableTTL "1"
```

>[!NOTE]
>
>This feature is avaiable with version **4.1.11** of the Dispatcher.

## Configuring Load Balancing - /statistics {#configuring-load-balancing-statistics}

The `/statistics` section defines categories of files for which Dispatcher scores the responsiveness of each render. Il dispatcher utilizza i punteggi per determinare quale rendering inviare una richiesta.

Ogni categoria creata definisce un pattern di glob. Il dispatcher confronta l&#39;URI del contenuto richiesto con questi pattern per determinare la categoria del contenuto richiesto:

* L&#39;ordine delle categorie determina l&#39;ordine in cui vengono confrontati con l&#39;URI.
* Il primo pattern di categoria che corrisponde all&#39;URI è la categoria del file. Non vengono valutati più pattern di categoria.

Il dispatcher supporta un massimo di 8 categorie statistiche. Se si definiscono più di 8 categorie, vengono utilizzati solo i primi 8.

**Rendering selezione**

Ogni volta che Dispatcher richiede una pagina sottoposta a rendering, utilizza il seguente algoritmo per selezionare il rendering:

1. If the request contains the render name in a `renderid` cookie, Dispatcher uses that render.
1. If the request includes no `renderid` cookie, Dispatcher compares the render statistics:

   1. Dispatcher determina la richiesta dell&#39;URI della richiesta.
   1. Il dispatcher determina quale rendering ha il punteggio risposta più basso per quella categoria e seleziona il rendering.

1. Se non viene ancora selezionato alcun rendering, utilizzate il primo rendering nell&#39;elenco.

Il punteggio per la categoria di un rendering si basa sui tempi di risposta precedenti, oltre che sulle connessioni precedenti non riuscite e con esito positivo che vengono eseguite dal dispatcher. Per ogni tentativo, la valutazione relativa alla categoria dell&#39;URI richiesto viene aggiornata.

>[!NOTE]
>
>Se non si utilizza il bilanciamento del carico, è possibile omettere questa sezione.

### Defining Statistics Categories {#defining-statistics-categories}

Definite una categoria per ogni tipo di documento per il quale desiderate conservare le statistiche per la selezione del rendering. La sezione /statistics contiene una sezione /categories. Per definire una categoria, aggiungi una riga sotto la sezione /categories che include il formato seguente:

`/name { /glob "pattern"}`

The category `name` must be unique to the farm. The `pattern` is described in the [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties) section.

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

The `/unavailablePenalty` property sets the time (in tenths of a second) that is applied to the render statistics when a connection to the render fails. Il dispatcher aggiunge l&#39;ora alla categoria statistiche che corrisponde all&#39;URI richiesto.

Ad esempio, la sanzione viene applicata quando la connessione TCP/IP al nome host o alla porta designata non può essere stabilita, poiché AEM non è in esecuzione (e non è ascolto) o a causa di un problema di rete.

The `/unavailablePenalty` property is a direct child of the `/farm` section (a sibling of the `/statistics` section).

If no `/unavailablePenalty` property exists, a value of &quot;1&quot; is used.

```xml
/unavailablePenalty "1"
```

## Identifying a Sticky Connection Folder - /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

The `/stickyConnectionsFor` property defines one folder that contains sticky documents; this will be accessed using the URL. Dispatcher invia tutte le richieste, da un singolo utente, che si trovano in questa cartella alla stessa istanza di rendering. Le connessioni fissi garantiscono che i dati delle sessioni siano presenti e coerenti per tutti i documenti. This mechanism uses the `renderid` cookie.

L&#39;esempio seguente definisce una connessione fisso alla cartella /products:

```xml
/stickyConnectionsFor "/products"
```

When a page is composed of content from several content nodes, include the `/paths` property that lists the paths to the content. For example, a page contains content from `/content/image`, `/content/video`, and `/var/files/pdfs`. La seguente configurazione abilita le connessioni fissi per tutto il contenuto della pagina:

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### httpOnly {#httponly}

When sticky connections are enabled, the dispatcher module sets the `renderid` cookie. This cookie doesn&#39;t have the `httponly` flag, which should be added in order to enhance security. You can do this by setting the `httpOnly` property in the `/stickyConnections` node of a `dispatcher.any` configuration file. The property&#39;s value (either 0 or 1) defines whether the `renderid` cookie has the `HttpOnly` attribute appended. Il valore predefinito è 0, che significa che l&#39;attributo non verrà aggiunto.

For additional information about the `httponly` flag, read [this page](https://www.owasp.org/index.php/HttpOnly).

### secure {#secure}

When sticky connections are enabled, the dispatcher module sets the `renderid` cookie. This cookie doesn&#39;t have the **secure** flag, which should be added in order to enhance security. You can do this by setting the `secure` property in the `/stickyConnections` node of a `dispatcher.any` configuration file. The property&#39;s value (either 0 or 1) defines whether the `renderid` cookie has the `secure` attribute appended. Il valore predefinito è 0, che significa che l&#39;attributo verrà aggiunto se * * la richiesta in arrivo è sicura. Se il valore è impostato su 1, il flag protetto verrà aggiunto a prescindere dal fatto che la richiesta in entrata sia sicura o meno.

## Handling Render Connection Errors {#handling-render-connection-errors}

Configura il comportamento del dispatcher quando il server di rendering restituisce un errore 500 o non è disponibile.

### Specifying a Health Check Page {#specifying-a-health-check-page}

Use the `/health_check` property to specify a URL that is checked when a 500 status code occurs. If this page also returns a 500 status code the instance is considered to be unavailable and a configurable time penalty ( `/unavailablePenalty`) is applied to the render before retrying.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifying the Page Retry Delay {#specifying-the-page-retry-delay}

The / `retryDelay` property sets the time (in seconds) that Dispatcher waits between rounds of connection attempts with the farm renders. Per ogni round, il numero massimo di volte in cui il dispatcher cerca una connessione a un rendering è il numero di rendering nella fattoria.

Dispatcher uses a value of `"1"` if `/retryDelay` is not explicitly defined. Nella maggior parte dei casi, il valore predefinito è appropriato.

```xml
/retryDelay "1"
```

### Configuring the Number of Retries {#configuring-the-number-of-retries}

The `/numberOfRetries` property sets the maximum number of rounds of connection attempts that Dispatcher performs with the renders. Se il dispatcher non riesce a connettersi a un rendering dopo questo numero di tentativi, il dispatcher restituisce una risposta non riuscita.

Per ogni round, il numero massimo di volte in cui il dispatcher cerca una connessione a un rendering è il numero di rendering nella fattoria. Therefore, the maximum number of times that Dispatcher attempts a connection is ( `/numberOfRetries`) x (the number of renders).

If the value is not explicitly defined, the default value is **5**.

```xml
/numberOfRetries "5"
```

### Using the Failover Mechanism {#using-the-failover-mechanism}

Abilitate il meccanismo di failover sulla fattoria del dispatcher per inviare nuovamente le richieste a diversi rendering quando la richiesta originale non riesce. Quando il failover è abilitato, il dispatcher ha il comportamento seguente:

* Quando una richiesta a un rendering restituisce lo stato HTTP 503 (NON DISPONIBILE), Dispatcher invia la richiesta a un altro rendering.
* When a request to a render returns HTTP status 50x (other than 503), Dispatcher sends a request for the page that is configured for the `health_check` property.

   * Se il controllo dello stato restituisce 500 (INTERNAL_ SERVER_ ERROR), Dispatcher invia la richiesta originale a un rendering diverso.
   * Se il controllo correttivo restituisce lo stato HTTP 200, il dispatcher restituisce l&#39;errore HTTP 500 iniziale al client.

Per abilitare il failover, aggiungete la riga seguente all&#39;azienda (o al sito Web):

```xml
/failover "1" 
```

>[!NOTE]
>
>To retry HTTP requests that contain a body, Dispatcher sends a `Expect: 100-continue` request header to the render before spooling the actual contents. CQ 5.5 con CQSE è quindi in grado di rispondere immediatamente a 100 (CONTINUA) o a un codice di errore. Anche altri contenitori servlet devono supportare questa funzione.

## Ignoring Interruption Errors - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>In genere questa opzione non è necessaria. Solo quando vengono visualizzati i messaggi di registro seguenti:
>
>`Error while reading response: Interrupted system call`

Any file system oriented system call can be interrupted `EINTR` if the object of the system call is located on a remote system accessed via NFS. Se queste chiamate del sistema possono essere ritirate o interrotte si basano su come il file system sottostante è stato montato sul computer locale.

Utilizzate il parametro /ignoreEINTR se l&#39;istanza ha una configurazione simile e il registro contiene il seguente messaggio:

`Error while reading response: Interrupted system call`

Internamente, il dispatcher legge la risposta dal server remoto (ovvero AEM) utilizzando un ciclo che può essere rappresentato come:

`while (response not finished) {  
read more data  
}`

Such messages can be generated when the `EINTR` occurs in the &quot; `read more data`&quot; section and are caused by the reception of a signal before any data was received.

To ignore such interrupts you can add the following parameter to `dispatcher.any` (before `/farms`):

`/ignoreEINTR "1"`

Setting `/ignoreEINTR` to `"1"` causes Dispatcher to continue to attempt to read data until the complete response is read. Il valore predefinito è 0 e disattiva l&#39;opzione.

## Designing Patterns for glob Properties {#designing-patterns-for-glob-properties}

Several sections in the Dispatcher configuration file use `glob` properties as selection criteria for client requests. I valori delle proprietà della glob sono pattern che il dispatcher confronta con un aspetto della richiesta, ad esempio il percorso della risorsa richiesta, o l&#39;indirizzo IP del client. For example, the items in the `/filter` section use glob patterns to identify the paths of the pages that Dispatcher acts on or rejects.

I valori di glob possono includere caratteri jolly e caratteri alfanumerici per definire il pattern.

| Carattere jolly | Descrizione | Esempi |
|--- |--- |--- |
| `*` | Corrisponde a zero o più istanze contigue di qualsiasi carattere della stringa. The final character of the match is determined by either of the following situations: <br/>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics:<br/><ul><li>Non a *</li><li>Not a?</li><li>Un carattere letterale (incluso uno spazio) o una classe di caratteri.</li><li>Viene raggiunta la fine del pattern.</li></ul>All&#39;interno di una classe di caratteri, il carattere viene interpretato letteralmente. | `*/geo*` Corrisponde a qualsiasi pagina sotto il `/content/geometrixx` nodo e il `/content/geometrixx-outdoors` nodo. The following HTTP requests match the glob pattern: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*`<br/>Corrisponde a qualsiasi pagina sotto il `/content/geometrixx-outdoors` nodo. For example, the following HTTP request matches the glob pattern: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Corrisponde a qualsiasi singolo carattere. Utilizzare classi di caratteri esterne. All&#39;interno di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*outdoors/??/*`<br/> Corrisponde alle pagine di qualsiasi lingua nel sito geometrixx-outdoors. For example, the following HTTP request matches the glob pattern: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>La seguente richiesta non corrisponde al pattern della glob: <br/><ul><li>&quot; GET /content/geometrixx-outdoors/en.html &quot;</li></ul> |
| `[ and ]` | Indica l&#39;inizio e la fine di una classe di caratteri. Le classi di caratteri possono includere uno o più intervalli di caratteri e singoli caratteri.<br/>Una corrispondenza si verifica se il carattere di destinazione corrisponde a qualsiasi carattere nella classe di caratteri o all&#39;interno di un intervallo definito.<br/>Se la parentesi quadre di chiusura non è inclusa, il pattern non genera corrispondenze. | `*[o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*`<br/>Corrisponde alle seguenti richieste HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Denota un intervallo di caratteri. Da utilizzare nelle classi di caratteri. Esterno a una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[m-p]men.html*` Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Does not match the following HTTP request:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Rifiuta la classe di caratteri o di caratteri che segue. Utilizzare solo per negare caratteri e intervalli di caratteri all&#39;interno delle classi di caratteri. Equivalent to the `^ wildcard`. <br/>Esterno a una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[!o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` o `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Rifiuta l&#39;intervallo di caratteri o caratteri che segue. Utilizzare per negare solo caratteri e intervalli di caratteri all&#39;interno delle classi di caratteri. Equivalent to the `!` wildcard character. <br/>Esterno a una classe di caratteri, questo carattere viene interpretato letteralmente. | The examples for the `!` wildcard character apply, substituting the `!` characters in the example patterns with `^` characters. |


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

## Logging {#logging}

Nella configurazione del server Web potete impostare:

* Posizione del file di registro del dispatcher.
* Il livello di registro.

Per ulteriori informazioni, consultate la documentazione del server Web e il file leggimi dell&#39;istanza di Dispatcher.

**Registri Apache ruotati/piè di pagina**

If using an **Apache** web server you can use the standard functionality for rotated and/or piped logs. Ad esempio, utilizzando i registri piè di pagina:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Questa operazione ruota automaticamente:

* il file di registro del dispatcher; con marca temporale nell&#39;estensione (logs/dispatcher. log % Y % m % d).
* settimanale (60 x 60 x 24 x 7 = 604800 secondi).

Please see the Apache web server documentation on Log Rotation and Piped Logs; for example [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>All&#39;installazione il livello di registro predefinito è elevato (ad es. livello 3 = Debug), in modo che il dispatcher registri tutti gli errori e gli avvisi. Questo è molto utile nelle fasi iniziali.
>
>However, this requires additional resources, so when the Dispatcher is working smoothly *according to your requirements*, you can(should) lower the log level.

### Trace Logging {#trace-logging}

Tra gli altri miglioramenti apportati al dispatcher, la versione 4.2.0 introduce anche Trace Logging.

Si tratta di un livello superiore rispetto alla registrazione di debug, con informazioni aggiuntive nei registri. Aggiunge l&#39;accesso per:

* I valori delle intestazioni inoltrate;
* La regola che viene applicata a una determinata azione.

You can enable Trace Logging by setting the log level to `4` in your web server.

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

## Confirming Basic Operation {#confirming-basic-operation}

Per confermare operazioni di base e interazione con il server Web, il dispatcher e l&#39;istanza di AEM potete utilizzare i seguenti passaggi:

1. Set the `loglevel` to `3`.

1. Avviare il server Web; viene avviato anche il dispatcher.
1. Avviate l&#39;istanza AEM.
1. Controllate i file di registro e di errore per il server Web e il dispatcher.\
   A seconda del server Web, dovresti visualizzare messaggi quali:\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   e:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Surf il sito Web tramite il server Web. Confermate che il contenuto sia visualizzato come richiesto.\
   For example, on a local installation where AEM runs on port `4502` and the web server on `80` access the Websites console using both:\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`I risultati devono essere identici. Confermate l&#39;accesso ad altre pagine con lo stesso meccanismo.

1. Verificate che la directory della cache sia in fase di compilazione.
1. Attivate una pagina per verificare che la cache sia stata scaricata correttamente.
1. If everything is operating correctly you can reduce the `loglevel` to `0`.

## Using Multiple Dispatchers {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più Dispatcher. Ad esempio, è possibile utilizzare:

* un dispatcher per pubblicare un sito Web nella rete Intranet
* un secondo Dispatcher, con un indirizzo diverso e con impostazioni di sicurezza diverse, per pubblicare lo stesso contenuto su Internet.

In tal caso, accertatevi che ogni richiesta sia attraversata da un solo dispatcher. Un dispatcher non gestisce le richieste derivanti da un altro dispatcher. Assicuratevi quindi che entrambi i dispatcher accedano direttamente al sito Web AEM.

## Debugging {#debugging}

When adding the header `X-Dispatcher-Info` to a request, Dispatcher answers whether the target was cached, returned from cached or not cacheable at all. The response header `X-Cache-Info` contains this information in a readable form. Potete utilizzare queste intestazioni di risposta per risolvere problemi relativi alle risposte memorizzate nella cache dal dispatcher.

This functionality is not enabled by default, so in order for the response header `X-Cache-Info` to be included, the farm must contain the following entry:

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

Also, the `X-Dispatcher-Info` header does not need a value, but if you use `curl` for testing you must supply a value in order to send the header, such as:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/we-retail/us/en.html
```

Below is a list containing the response headers that `X-Dispatcher-Info` will return:

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
   Il modello nome file temporaneo supera il nome file più lungo possibile sul sistema. Prima di creare o sovrascrivere il file memorizzato nella cache, il dispatcher crea innanzitutto un file temporaneo. The temporary file name is the target file name with the characters `_YYYYXXXXXX` appended to it, where the `Y` and `X` will be replaced to create a unique name.
* **non cachable: l&#39;URL della richiesta non ha estensione**\
   The request URL has no extension, or there is a path following the file extension, for example: `/test.html/a/path`.
* **non cachable: la richiesta non è GET o HEAD**
Il metodo HTTP non è né GET né HEAD. Il dispatcher presuppone che l&#39;output contenga dati dinamici che non dovrebbero essere memorizzati nella cache.
* **non cachable: request contains a query string**\
   La richiesta conteneva una stringa di query. Il dispatcher presuppone che l&#39;output dipenda dalla stringa query specificata e quindi non la cache.
* **non cachable: session manager did&#39;t authentof**\
   The farm&#39;s cache is governed by a session manager (the configuration contains a `sessionmanagement` node) and the request didn&#39;t contain the appropriate authentication information.
* **non cachable: la richiesta contiene l&#39;autorizzazione**\
   The farm is not allowed to cache output ( `allowAuthorized 0`) and the request contains authentication information.
* **non cachable: target è una directory**\
   Il file di destinazione è una directory. This might point to some conceptual mistake, where a URL and some sub-URL both contain cacheable output, for example if a request to `/test.html/a/file.ext` comes first and contains cacheable output, the dispatcher will not be able to cache the output of a subsequent request to `/test.html`.
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
