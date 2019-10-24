---
title: Configurazione di Dispatcher
seo-title: Configurazione di Dispatcher
description: Scopri come configurare il dispatcher.
seo-description: Scopri come configurare il dispatcher.
uuid: 253ef0f7-2491-4set-ab22-97439df29fd6
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: '1193211344162'
topic-tags: dispatcher
content-type: reference
discoiquuid: aeffee8e-bb34-42a7-9a5e-b7d0e848391a
translation-type: tm+mt
source-git-commit: eed7c3f77ec64f2e7c5cfff070ef96108886a059

---


# Configurazione di Dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

Nelle sezioni seguenti viene descritto come configurare vari aspetti del dispatcher.

## Supporto per IPv4 e IPv6 {#support-for-ipv-and-ipv}

Tutti gli elementi di AEM e Dispatcher possono essere installati nelle reti IPv4 e IPv6. Consultate [IPV4 e IPV6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes).

## File di configurazione del dispatcher {#dispatcher-configuration-files}

Per impostazione predefinita, la configurazione del dispatcher è memorizzata nel file di `dispatcher.any` testo, anche se è possibile modificare il nome e la posizione del file durante l'installazione.

Il file di configurazione contiene una serie di proprietà a valore singolo o multivalore che controllano il comportamento del dispatcher:

* I nomi delle proprietà hanno il prefisso barra `/`.
* Le proprietà multivalore racchiudono elementi secondari utilizzando le parentesi `{ }`.

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

* Se il file di configurazione è grande, potete suddividerlo in diversi file più piccoli (che sono più facili da gestire) e quindi includerli.
* Per includere i file generati automaticamente.

Ad esempio, per includere il file myFarm.any nella configurazione /farm, utilizza il seguente codice:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Utilizzate l'asterisco ("*") come carattere jolly per specificare un intervallo di file da includere.

Ad esempio, se i file `farm_1.any` fino a `farm_5.any` contenere la configurazione di farm da uno a cinque, è possibile includerli come segue:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Utilizzo delle variabili di ambiente {#using-environment-variables}

È possibile utilizzare le variabili di ambiente nelle proprietà con valori stringa nel file dispatcher.any anziché codificare i valori. Per includere il valore di una variabile di ambiente, utilizzare il formato `${variable_name}`.

Ad esempio, se il file dispatcher.any si trova nella stessa directory della directory della cache, è possibile utilizzare il seguente valore per la proprietà [docroot](dispatcher-configuration.md#main-pars-title-30) :

```xml
/docroot "${PWD}/cache"
```

Come altro esempio, se create una variabile di ambiente denominata `PUBLISH_IP` che memorizza il nome host dell’istanza di pubblicazione AEM, potete utilizzare la seguente configurazione della proprietà [/renders](dispatcher-configuration.md#main-pars-127-25-0008) :

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Denominazione dell’istanza del dispatcher {#naming-the-dispatcher-instance-name}

Utilizzare la `/name` proprietà per specificare un nome univoco per identificare l'istanza del Dispatcher. La `/name` proprietà è una proprietà di livello principale nella struttura di configurazione.

## Definizione di farm {#defining-farms-farms}

La `/farms` proprietà definisce uno o più set di comportamenti Dispatcher, in cui ogni set è associato a siti Web o URL diversi. La `/farms` proprietà può includere una o più farm:

* Utilizzate un'unica farm quando desiderate che il dispatcher gestisca tutte le pagine Web o i siti Web nello stesso modo.
* Creare più farm quando aree diverse del sito Web o siti Web diversi richiedono un comportamento diverso da quello del dispatcher.

La `/farms` proprietà è una proprietà di livello principale nella struttura di configurazione. Per definire una farm, aggiungete una proprietà figlio alla `/farms` proprietà. Utilizzate un nome di proprietà che identifichi in modo univoco la farm all'interno dell'istanza Dispatcher.

La `/farmname` proprietà è multivalore e contiene altre proprietà che definiscono il comportamento del dispatcher:

* URL delle pagine a cui si applica la farm.
* Uno o più URL di servizio (in genere istanze di pubblicazione AEM) da utilizzare per il rendering dei documenti.
* Statistiche da utilizzare per il bilanciamento del carico di più renderer di documenti.
* Diversi altri comportamenti, ad esempio quali file memorizzare nella cache e dove.

Il valore può includere qualsiasi carattere alfanumerico (a-z, 0-9). L'esempio seguente mostra la definizione dello scheletro per due farm denominate `/daycom` e `/docsdaycom`:

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
>Se utilizzate più di una farm di rendering, l'elenco viene valutato dal basso verso l'alto. Ciò è particolarmente importante quando si definiscono gli host [](dispatcher-configuration.md#main-pars-117-15-0006) virtuali per i siti Web.

Ogni proprietà farm può contenere le seguenti proprietà figlio:

| Nome proprietà | Descrizione |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | Homepage predefinita (facoltativo) (solo IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | Intestazioni dalla richiesta HTTP client da trasmettere. |
| [/virtualhosts](#identifying-virtual-hosts-virtual-hosts) | Host virtuali per la farm. |
| [/session management](#enabling-secure-sessions-session-management) | Supporto per la gestione e l’autenticazione delle sessioni. |
| [/renderer](#defining-page-renderers-renders) | I server che forniscono le pagine di cui è stato eseguito il rendering (in genere istanze di pubblicazione AEM). |
| [/filter](#configuring-access-to-content-filter) | Definisce gli URL a cui il dispatcher consente l'accesso. |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | Configura l’accesso agli URL personalizzati. |
| [/propagateSyndPost](#forwarding-syndication-requests-propagate-syndpost) | Supporto per l'inoltro delle richieste di sindacazione. |
| [/cache](#configuring-the-dispatcher-cache-cache) | Configura il comportamento nella cache. |
| [/statistics](#configuring-load-balancing-statistics) | Definizione delle categorie statistiche per i calcoli di bilanciamento del carico. |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-sticky-connections-for) | Cartella contenente documenti fissi. |
| [/health_check](#specifying-a-health-check-page) | URL da utilizzare per determinare la disponibilità del server. |
| [/tryDelay](#specifying-the-page-retry-delay) | Il ritardo prima di riprovare a eseguire una connessione non riuscita. |
| [/non disponibilePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | Sanzioni che influiscono sulle statistiche per i calcoli di bilanciamento del carico. |
| [/failover](#using-the-fail-over-mechanism) | Inviate di nuovo le richieste a diversi rendering quando la richiesta originale non riesce. |
| [/auth_checker](permissions-cache.md) | Per il caching sensibile alle autorizzazioni, consultate [Memorizzazione in cache di contenuto](permissions-cache.md)protetto. |

## Specificare una pagina predefinita (solo IIS) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>Il `/homepage`parametro (solo IIS) non funziona più. Utilizzare invece il modulo [di riscrittura URL](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)IIS.
>
>Se si utilizza Apache, è necessario utilizzare il `mod_rewrite` modulo. Per informazioni su `mod_rewrite` (ad esempio, [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)), consultate la documentazione del sito Web Apache. Quando si utilizza `mod_rewrite`, è consigliabile utilizzare il flag **['passthrough|PT' (passare attraverso il gestore successivo)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** per forzare il motore di riscrittura a impostare il `uri` campo della `request_rec` struttura interna sul valore del `filename` campo.

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

## Specifica delle intestazioni HTTP da trasmettere {#specifying-the-http-headers-to-pass-through-clientheaders}

La `/clientheaders` proprietà definisce un elenco di intestazioni HTTP che il dispatcher passa dalla richiesta HTTP client al renderer (istanza AEM).

Per impostazione predefinita, il dispatcher inoltra le intestazioni HTTP standard all’istanza AEM. In alcuni casi, potrebbe essere necessario inoltrare intestazioni aggiuntive o rimuovere intestazioni specifiche:

* Aggiungete intestazioni, come intestazioni personalizzate, che l’istanza AEM prevede nella richiesta HTTP.
* Rimuovete le intestazioni, ad esempio le intestazioni di autenticazione, rilevanti solo per il server Web.

Se personalizzate il set di intestazioni da scorrere, dovete specificare un elenco completo di intestazioni, incluse quelle normalmente incluse per impostazione predefinita.

Ad esempio, un'istanza Dispatcher che gestisce le richieste di attivazione della pagina per le istanze pubblicate richiede l' `PATH` intestazione della `/clientheaders` sezione. L' `PATH` intestazione consente la comunicazione tra l'agente di replica e il dispatcher.

Di seguito è riportato un esempio di configurazione per `/clientheaders`:

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

## Identificazione degli host virtuali {#identifying-virtual-hosts-virtualhosts}

La `/virtualhosts` proprietà definisce un elenco di tutte le combinazioni di nome host/URI accettate dal Dispatcher per questa farm. È possibile utilizzare il carattere asterisco ("*") come carattere jolly. I valori per la proprietà / `virtualhosts` utilizzano il formato seguente:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Facoltativo) `https://` oppure `https://.`
* `host`: Nome o indirizzo IP del computer host e, se necessario, numero di porta. (Vedere [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`: (Facoltativo) Percorso delle risorse.

La seguente configurazione di esempio gestisce le richieste per i domini .com e .ch della mia azienda e per tutti i domini di mySubDivision:

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

### Risoluzione dell'host virtuale {#resolving-the-virtual-host}

Quando il dispatcher riceve una richiesta HTTP o HTTPS, trova il valore host virtuale che meglio corrisponde alle `host,` intestazioni e `uri`alle `scheme` intestazioni della richiesta. Il dispatcher valuta i valori delle `virtualhosts` proprietà nel seguente ordine:

* Il dispatcher inizia dalla farm più bassa e procede verso l’alto nel file dispatcher.any.
* Per ogni farm, Dispatcher inizia con il valore superiore nella `virtualhosts` proprietà e procede verso il basso fino all'elenco dei valori.

Dispatcher trova il valore host virtuale più simile nel modo seguente:

* Viene utilizzato il primo host virtuale rilevato che corrisponde a tutti e tre `host`, il `scheme`e `uri` il numero della richiesta.
* Se nessun `virtualhosts` valore ha `scheme` e `uri` parti che corrispondono sia alla richiesta che `scheme` all'host virtuale rilevato per la prima volta che corrisponde alla `uri` `host` richiesta, viene utilizzato l'host virtuale rilevato per la prima volta.
* Se non `virtualhosts` esiste una parte host che corrisponda all'host della richiesta, viene utilizzato l'host virtuale superiore della farm superiore.

Pertanto, è necessario posizionare l'host virtuale predefinito nella parte superiore della `virtualhosts` proprietà nella farm superiore del file dispatcher.any.

### Esempio di risoluzione host virtuale {#example-virtual-host-resolution}

L'esempio seguente rappresenta uno snippet di un file dispatcher.any che definisce due farm Dispatcher e ogni farm definisce una `virtualhosts` proprietà.

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

Utilizzando questo esempio, la tabella seguente mostra gli host virtuali risolti per le richieste HTTP indicate:

| URL richiesta | Host virtuale risolto |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/*;` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## Abilitazione di sessioni protette - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` Per attivare questa funzione, **è necessario** impostarla `"0"` nella `/cache` sezione.

Create una sessione protetta per l'accesso alla farm di rendering in modo che gli utenti debbano accedere a qualsiasi pagina della farm. Dopo l'accesso, gli utenti possono accedere alle pagine della farm. Consultate [Creazione di un gruppo](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) utenti chiuso per informazioni sull’utilizzo di questa funzione con i CUG. Inoltre, vedere l'elenco di controllo [della](/help/using/security-checklist.md) sicurezza del dispatcher prima di iniziare a visualizzare in diretta.

La `/sessionmanagement` proprietà è una sottoproprietà di `/farms`.

>[!CAUTION]
>
>Se le sezioni del sito Web utilizzano requisiti di accesso diversi, è necessario definire più farm.

**/sessionmanagement** ha diversi sottoparametri:

**/directory** (obbligatorio)

La directory in cui sono memorizzate le informazioni della sessione. Se la directory non esiste, viene creata.

>[!CAUTION]
>
> Durante la configurazione del sottoparametro della directory **non** puntare alla cartella principale (`/directory "/"`) in quanto può causare gravi problemi. Dovete sempre specificare il percorso della cartella in cui sono memorizzate le informazioni della sessione. Esempio:

```xml
/sessionmanagement 
  { 
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (facoltativo)

Modalità di codifica delle informazioni sulla sessione. Utilizzate "md5" per la cifratura utilizzando l'algoritmo md5, o "hex" per la codifica esadecimale. Se crittografate i dati della sessione, un utente con accesso al file system non sarà in grado di leggere il contenuto della sessione. Il valore predefinito è "md5".

**/header** (facoltativo)

Nome dell’intestazione HTTP o del cookie in cui vengono memorizzate le informazioni di autorizzazione. Se memorizzate le informazioni nell’intestazione http, utilizzate `HTTP:<*header-name*>`. Per memorizzare le informazioni in un cookie, utilizzare `Cookie:<header-name>`. Se non si specifica un valore `HTTP:authorization` viene utilizzato.

**/timeout** (facoltativo)

Il numero di secondi fino al timeout della sessione dopo l’ultimo utilizzo. Se non viene specificato "800", la sessione scadrà poco più di 13 minuti dopo l’ultima richiesta dell’utente.

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

## Definizione dei renderer di pagina {#defining-page-renderers-renders}

La proprietà /renders definisce l'URL al quale il dispatcher invia le richieste per eseguire il rendering di un documento. La sezione di esempio seguente `/renders` identifica una singola istanza di AEM per il rendering:

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

La seguente sezione /renders di esempio identifica un’istanza AEM che viene eseguita sullo stesso computer del Dispatcher:

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

La seguente sezione di esempio /renders distribuisce le richieste di rendering tra due istanze AEM:

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

### Opzioni di rendering {#renders-options}

**/timeout**

Specifica il timeout di connessione per l'accesso all'istanza di AEM, in millisecondi. Il valore predefinito è "0", che causerà un'attesa indefinita del dispatcher.

**/receiveTimeout**

Specifica il tempo in millisecondi che una risposta può richiedere. Il valore predefinito è "600000" e il Dispatcher deve attendere 10 minuti. L'impostazione "0" elimina completamente il timeout.\
Se viene raggiunto il timeout durante l'analisi delle intestazioni di risposta, viene restituito uno stato HTTP 504 (gateway non valido). Se il timeout viene raggiunto durante la lettura del corpo della risposta, il dispatcher restituirà la risposta incompleta al client, ma eliminerà tutti i file della cache eventualmente scritti.

**/ipv4**

Specifica se Dispatcher utilizza la `getaddrinfo` funzione (per IPv6) o la `gethostbyname` funzione (per IPv4) per ottenere l'indirizzo IP del rendering. Viene utilizzato il valore 0. `getaddrinfo` È possibile `gethostbyname` utilizzare il valore 1. Il valore predefinito è 0.

La funzione getaddrinfo restituisce un elenco di indirizzi IP. Dispatcher esegue un'iterazione dell'elenco di indirizzi finché non stabilisce una connessione TCP/IP. Pertanto, la proprietà ipv4 è importante quando il nome host di rendering è associato a più indirizzi IP e l'host, in risposta alla funzione getaddrinfo, restituisce un elenco di indirizzi IP che sono sempre nello stesso ordine. In questa situazione, è necessario utilizzare la funzione gethostbyname in modo che l'indirizzo IP con cui il Dispatcher si collega sia casuale.

Amazon Elastic Load Balancing (ELB) è un servizio che risponde a getaddrinfo con un elenco potenzialmente uguale di indirizzi IP.

**/secure**

Se il valore della `/secure` proprietà è "1", il dispatcher utilizza HTTPS per comunicare con l'istanza AEM. Per ulteriori dettagli, consultate anche [Configurazione del dispatcher per l’utilizzo di SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Con la versione **4.1.6** del dispatcher, puoi configurare la `/always-resolve` proprietà come segue:

* Se impostato su "1", il nome host verrà risolto su ogni richiesta (il dispatcher non memorizzerà mai nella cache alcun indirizzo IP). Potrebbe verificarsi un leggero impatto sulle prestazioni a causa della chiamata aggiuntiva necessaria per ottenere le informazioni sull'host per ogni richiesta.
* Se la proprietà non è impostata, l'indirizzo IP verrà memorizzato nella cache per impostazione predefinita.

Questa proprietà può essere utilizzata anche in caso di problemi di risoluzione IP dinamica, come illustrato nell'esempio seguente:

```xml
/renders {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## Configurazione dell'accesso al contenuto {#configuring-access-to-content-filter}

Utilizzate la `/filter` sezione per specificare le richieste HTTP accettate da Dispatcher. Tutte le altre richieste vengono inviate al server Web con un codice di errore 404 (pagina non trovata). Se non esiste alcuna `/filter` sezione, tutte le richieste vengono accettate.

**** Nota: Le richieste per il [file](dispatcher-configuration.md#main-pars-title-28) di stato vengono sempre rifiutate.

>[!CAUTION]
>
>Per ulteriori considerazioni su come limitare l'accesso tramite Dispatcher, consulta l'elenco di controllo [Protezione dispatcher](security-checklist.md) . Inoltre, consultate l'elenco di controllo [AEM Security](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) per ulteriori dettagli di sicurezza relativi all'installazione di AEM.

La sezione /filter è composta da una serie di regole che negano o consentono l’accesso al contenuto in base ai pattern presenti nella parte della riga della richiesta HTTP. È consigliabile utilizzare una strategia whtelist per la sezione /filter:

* In primo luogo, negare l'accesso a tutto.
* Consentire l'accesso al contenuto in base alle esigenze.

### Definizione di un filtro {#defining-a-filter}

Ogni elemento della `/filter` sezione include un tipo e un pattern associati a un elemento specifico della riga della richiesta o all'intera riga della richiesta. Ogni filtro può contenere i seguenti elementi:

* **Tipo**: Indica `/type` se consentire o negare l'accesso alle richieste che corrispondono al pattern. Il valore può essere `allow` o `deny`.

* **** Elemento della linea di richiesta: Includete `/method`, `/url`, `/query`o `/protocol` un pattern per filtrare le richieste in base a queste parti specifiche della parte della riga di richiesta della richiesta HTTP. Il filtraggio sugli elementi della riga della richiesta (anziché sull’intera riga della richiesta) è il metodo di filtro preferito.

* **** Elementi avanzati della riga richiesta: A partire da Dispatcher 4.2.0, sono disponibili quattro nuovi elementi filtro. Questi nuovi elementi sono `/path`, `/selectors`, `/extension` e `/suffix` rispettivamente. Includete uno o più di questi elementi per controllare ulteriormente i pattern URL.

>[!NOTE]
>
>Per ulteriori informazioni sulla parte della riga di richiesta a cui fanno riferimento ciascuno di questi elementi, consultate la pagina wiki [Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) (Decomposizione URL slm).

* **Proprietà** elemento: La `/glob` proprietà viene utilizzata per corrispondere all'intera riga di richiesta della richiesta HTTP.

>[!CAUTION]
>
>Il filtraggio con i globs non è più supportato in Dispatcher. In quanto tale, si dovrebbe evitare di utilizzare globs nelle `/filter` sezioni, in quanto potrebbe portare a problemi di sicurezza. Pertanto, anziché:

`/glob "* *.css *"`

dovrebbe utilizzare

`/url "*.css"`

#### Parte della riga di richiesta delle richieste HTTP {#the-request-line-part-of-http-requests}

HTTP/1.1 definisce la riga [di](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) richiesta come segue:

*Richiesta metodo - URI HTTP-Version*&lt;CRLF&gt;

I caratteri &lt;CRLF&gt; rappresentano un ritorno a capo seguito da un avanzamento riga. L’esempio seguente è la riga-richiesta ricevuta quando un cliente richiede la pagina en del sito Geometrixx-Outoors:

GET /content/geometrixx-outdoors/en.html HTTP.1.1&lt;CRLF&gt;

I pattern devono tenere conto degli spazi nella riga di richiesta e dei caratteri &lt;CRLF&gt;.

#### virgolette doppie o virgolette singole {#double-quotes-vs-single-quotes}

Quando create le regole del filtro, utilizzate virgolette doppie `"pattern"` per i pattern semplici. Se si utilizza Dispatcher 4.2.0 o versione successiva e il pattern include un'espressione regolare, è necessario racchiudere il pattern regex `'(pattern1|pattern2)'` tra virgolette singole.

#### Regular Expressions {#regular-expressions}

Dopo il Dispatcher 4.2.0, potete includere nei modelli di filtro anche le espressioni regolari POSIX Extended.

#### Risoluzione dei problemi relativi ai filtri {#troubleshooting-filters}

Se i filtri non si attivano come previsto, abilita [Trace Logging](#trace-logging) sul dispatcher per vedere quale filtro intercetta la richiesta.

#### Esempio di filtro: Rifiuta tutto {#example-filter-deny-all}

Nella sezione seguente del filtro di esempio il dispatcher nega le richieste per tutti i file. È necessario negare l'accesso a tutti i file e quindi consentire l'accesso ad aree specifiche.

```xml
  /0001  { /glob "*" /type "deny" }
```

Le richieste a un'area negata in modo esplicito causano la restituzione di un codice di errore 404 (pagina non trovata).

#### Esempio di filtro: Rifiuta accesso a aree specifiche {#example-filter-deny-acess-to-specific-areas}

I filtri consentono inoltre di negare l’accesso a vari elementi, ad esempio pagine ASP e aree sensibili all’interno di un’istanza di pubblicazione. Il filtro seguente nega l'accesso alle pagine ASP:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Esempio di filtro: Abilita richieste POST {#example-filter-enable-post-requests}

Il seguente filtro di esempio consente di inviare i dati del modulo tramite il metodo POST:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Esempio di filtro: Consenti accesso alla console Flusso di lavoro {#example-filter-allow-access-to-the-workflow-console}

L’esempio seguente mostra un filtro utilizzato per negare l’accesso esterno alla console Flusso di lavoro:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

Se l’istanza di pubblicazione utilizza un contesto applicazione Web (ad esempio Pubblica), questo può essere aggiunto anche alla definizione del filtro.

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

Se è comunque necessario accedere a singole pagine all'interno dell'area limitata, è possibile consentirne l'accesso. Ad esempio, per consentire l’accesso alla scheda Archivio nella console Flusso di lavoro, aggiungi la sezione seguente:

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>Quando più pattern di filtri si applicano a una richiesta, l'ultimo pattern di filtri applicato è valido.

#### Esempio di filtro: Utilizzo di espressioni regolari {#example-filter-using-regular-expressions}

Questo filtro abilita le estensioni nelle directory di contenuto non pubblico utilizzando un'espressione regolare, definita qui tra virgolette singole:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Esempio di filtro: Filtrare elementi aggiuntivi di un URL richiesta {#example-filter-filter-additional-elements-of-a-request-url}

Di seguito è riportato un esempio di regola che blocca l’acquisizione del contenuto dal `/content` percorso e dalla struttura ad albero secondaria, utilizzando filtri per percorso, selettori ed estensioni:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### Esempio /filter, sezione {#example-filter-section}

Durante la configurazione del dispatcher, devi limitare il più possibile l'accesso esterno. L'esempio seguente fornisce un accesso minimo per i visitatori esterni:

* `/content`
* contenuti vari, quali progetti e librerie di clienti; ad esempio:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

Dopo aver creato i filtri, [verificate l’accesso](dispatcher-configuration.md#main-pars-title-19) alla pagina per garantire la protezione dell’istanza AEM.

La seguente sezione /filter del file dispatcher.any può essere utilizzata come base nel file di configurazione [del](dispatcher-configuration.md) dispatcher.

Questo esempio è basato sul file di configurazione predefinito fornito con Dispatcher e destinato ad essere utilizzato in un ambiente di produzione. Gli elementi con il prefisso # vengono disattivati (commentato), è necessario prestare attenzione se si decide di attivarli (rimuovendo il simbolo # su quella riga) in quanto questo può avere un impatto sulla sicurezza.

È necessario negare l'accesso a tutto, quindi consentire l'accesso a elementi specifici (limitati):

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
>Se utilizzato con Apache, progettate i pattern dell’URL del filtro in base alla proprietà DispatcherUseProcessedURL del modulo Dispatcher. (Vedete Server Web [Apache - Configurare il server Web Apache per il dispatcher](dispatcher-install.md#main-pars-55-35-1022).)

>[!NOTE]
>
>I filtri 0030 e 0031 per gli elementi multimediali dinamici sono applicabili ad AEM 6.0 e versioni successive.

Se scegliete di estendere l'accesso, tenete presenti le seguenti raccomandazioni:

* Se utilizzate CQ versione 5.4 o una versione precedente, l’accesso esterno a `/admin` deve essere sempre *completamente* disattivato.

* Quando consentite l’accesso ai file in `/libs`, prestate attenzione. L'accesso dovrebbe essere consentito su base individuale.
* Negare l'accesso alla configurazione di replica in modo che non possa essere visibile:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Rifiuta l'accesso al proxy inverso Google Gadget:

   * `/libs/opensocial/proxy*`

A seconda dell’installazione, potrebbero essere disponibili risorse aggiuntive in `/libs`, `/apps` o altrove. Potete utilizzare il `access.log` file come metodo per determinare le risorse a cui si accede esternamente.

>[!CAUTION]
>
>L'accesso alle console e alle directory può rappresentare un rischio per gli ambienti di produzione. A meno che non si disponga di giustificazioni esplicite, devono rimanere disattivati (commentato).

>[!CAUTION]
>
>Se [utilizzate i rapporti in un ambiente](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) di pubblicazione, configurate il dispatcher in modo da negare l’accesso ai visitatori `/etc/reports` per esterni.

### Limitazione delle stringhe di query {#restricting-query-strings}

Dalla versione 4.1.5 del dispatcher, utilizzare la `/filter` sezione per limitare le stringhe di query. Si consiglia vivamente di consentire in modo esplicito le stringhe di query ed escludere la tolleranza generica attraverso gli elementi `allow` filtro.

Una singola voce può avere *un* elemento di stile *o una combinazione di* metodo *,* url *,* query *e* versione, ma non entrambi. L'esempio seguente consente la stringa di `a=*` query e nega tutte le altre stringhe di query per gli URL che si risolvono nel `/etc` nodo:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Se una regola contiene una `/query`, corrisponderà solo alle richieste che contengono una stringa di query e che corrispondono al pattern di query fornito.
>
>Nell'esempio precedente, se le richieste a `/etc` cui non è associata una stringa di query devono essere consentite, sono necessarie le seguenti regole:


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Verifica della protezione del dispatcher {#testing-dispatcher-security}

I filtri del dispatcher devono bloccare l'accesso alle pagine e agli script seguenti sulle istanze di pubblicazione AEM. Usate un browser Web per tentare di aprire le pagine seguenti come un visitatore del sito e verificare che venga restituito un codice 404. Se si ottiene un altro risultato, regolate i filtri.

Il rendering normale della pagina deve essere visualizzato per /content/add_valid_page.html?debug=layout.


* /admin
* /system/console
* /dav/crx.default
* /crx
* /bin/crxde/logs
* /jcr:system/jcr:versionStorage.json
* /_jcr_system/_jcr_versionStorage.json
* /libs/wcm/core/content/siteadmin.html
* /libs/collab/core/content/admin.html
* /libs/cq/ui/content/dumplibs.html
* /var/linkchecker.html
* /etc/linkchecker.html
* /home/users/a/admin/profile.json
* /home/users/a/admin/profile.xml
* /libs/cq/core/content/login.json
* ../libs/foundation/components/text/text.jsp
* /content/.{.}
* /apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata
* /libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet
* /content.pages.json
* /content.languages.json
* /content.blueprint.json
* /content.-1.json
* /content.10.json
* /content.infinity.json
* /content.tidy.json
* /content.tidy.-1.blubber.json
* /content/dam.tidy.-100.json
* /content/content/geometrixx.sitemap.txt
* /content/add_valid_page.query.json?Statement=/*
* /content/add_valid_page.qu%65ry.js%6Fn?istruzione=/*
* /content/add_valid_page.query.json?istruzione=/*[@TransportPassword]/(@TransportPassword%20|%20@transportUri%20|%20@transportUser)
* /content/add_valid_path_to_a_page/_jcr_content.json
* /content/add_valid_path_to_a_page/jcr:content.json
* /content/add_valid_path_to_a_page/_jcr_content.feed
* /content/add_valid_path_to_a_page/jcr:content.feed
* /content/add_valid_path_to_a_page/nomepagina._jcr_content.feed
* /content/add_valid_path_to_a_page/pagename.jcr:content.feed
* /content/add_valid_path_to_a_page/pagename.docview.xml
* /content/add_valid_path_to_a_page/pagename.docview.json
* /content/add_valid_path_to_a_page/pagename.sysview.xml
* /etc.xml
* /content.feed.xml
* /content.rss.xml
* /content.feed.html
* /content/add_valid_page.html?debug=layout
* /projects
* /tagging
* /etc/replication.html
* /etc/cloudservices.html
* /benvenuti

Emettere il comando seguente in un terminale o in un prompt dei comandi per determinare se l'accesso in scrittura anonimo è abilitato. Non dovresti essere in grado di scrivere dati sul nodo.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Emettere il seguente comando in un terminale o in un prompt dei comandi per tentare di annullare la validità della cache del Dispatcher e assicurarsi di ricevere una risposta di codice 404:

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## Abilitazione dell'accesso agli URL personalizzati {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

Configurate il dispatcher per abilitare l’accesso agli URL personalizzati configurati per le pagine CQ o AEM.

Quando l’accesso agli URL personalizzati è abilitato, Dispatcher chiama periodicamente un servizio in esecuzione sull’istanza di rendering per ottenere un elenco di URL personalizzati. Il dispatcher memorizza l'elenco in un file locale. Quando una richiesta di pagina viene rifiutata a causa di un filtro nella `/filter` sezione , Dispatcher consulta l’elenco degli URL personalizzati. Se l’URL negato è presente nell’elenco, il dispatcher consente l’accesso all’URL personalizzato.

Per abilitare l’accesso agli URL personalizzati, aggiungete una `/vanity_urls` sezione alla `/farms` sezione, simile al seguente esempio:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

La `/vanity_urls` sezione contiene le proprietà seguenti:

* `/url`: Percorso del servizio URL personalizzato in esecuzione sull’istanza di rendering. Il valore di questa proprietà deve essere `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`: Il percorso del file locale in cui il dispatcher memorizza l’elenco degli URL personalizzati. Verificate che il dispatcher disponga dell'accesso in scrittura a questo file.
* `/delay`: (Secondi) L'intervallo tra le chiamate al servizio URL personalizzato.

>[!NOTE]
>
>Se il rendering è un’istanza di AEM, è necessario installare il pacchetto [VanityURLS-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) per installare il servizio URL personalizzato. Consultate [Accesso a Package Share](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare).

Utilizzate la procedura seguente per abilitare l'accesso agli URL personalizzati.

1. Se il servizio di rendering è un’istanza di AEM, installate il pacchetto com.adobe.granite.dispatcher.vanityurl.content sull’istanza di pubblicazione (consultate la nota precedente).
1. Per ogni URL personalizzato configurato per una pagina AEM o CQ, accertatevi che la ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` configurazione neghi l’URL. Se necessario, aggiungete un filtro che neghi l’URL.
1. Aggiungete la `/vanity_urls` sezione seguente `/farms`.
1. Riavviate il server Web Apache.

## Inoltro delle richieste di sindacazione - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

Le richieste di sindacazione sono in genere destinate solo al dispatcher, pertanto per impostazione predefinita non vengono inviate al renderer (ad esempio, un’istanza di AEM).

Se necessario, impostare la proprietà /propagateSyndPost su "1" per inoltrare le richieste di sindacazione al Dispatcher. Se impostato, accertatevi che le richieste POST non vengano negate nella sezione del filtro.

## Configurazione della cache del dispatcher - /cache {#configuring-the-dispatcher-cache-cache}

La `/cache` sezione controlla come il dispatcher memorizza nella cache i documenti. Configurate diverse sottoproprietà per implementare le strategie di caching:


* /docroot
* /statfile
* /serveStaleOnError
* /allowAuthorized
* /regole
* /statfileslevel
* /invalidate
* /invalidateHandler
* /allowClients
* /ignoreUrlParams
* /header
* /mode
* /GracePeriod
* /enableTTL


Esempio di sezione della cache:

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
>Per il caching sensibile alle autorizzazioni, leggete [Memorizzazione del contenuto](permissions-cache.md)protetto nella cache.

### Specifica della directory della cache {#specifying-the-cache-directory}

La `/docroot` proprietà identifica la directory in cui sono memorizzati i file memorizzati nella cache.

>[!NOTE]
>
>Il valore deve corrispondere esattamente al percorso della radice del documento del server Web, in modo che il dispatcher e il server Web gestiscano gli stessi file.\
>Il server Web è responsabile della distribuzione del codice di stato corretto quando viene utilizzato il file della cache del dispatcher, per questo è importante che possa trovarlo.

Se si utilizzano più farm, ogni farm deve utilizzare un documento principale diverso.

### Denominazione del file di stato {#naming-the-statfile}

La `/statfile` proprietà identifica il file da utilizzare come file di stato. Il dispatcher utilizza questo file per registrare l'ora dell'aggiornamento del contenuto più recente. Il file di stato può essere un qualsiasi file sul server Web.

Il file di stato non ha contenuto. Quando il contenuto viene aggiornato, il dispatcher aggiorna la marca temporale. Il file di stato predefinito è denominato .stat e viene memorizzato nel docroot. Il dispatcher blocca l'accesso al file di stato.

>[!NOTE]
>
>Se `/statfileslevel` è configurata, Dispatcher ignora la `/statfile` proprietà e utilizza .stat come nome.

### Trasmissione di documenti non aggiornati in caso di errori {#serving-stale-documents-when-errors-occur}

La `/serveStaleOnError` proprietà controlla se il dispatcher restituisce documenti invalidati quando il server di rendering restituisce un errore. Per impostazione predefinita, quando un file di stato viene toccato e invalida il contenuto memorizzato nella cache, il dispatcher elimina il contenuto memorizzato nella cache al successivo richiamo.

Se `/serveStaleOnError` è impostato su "1", il dispatcher non elimina il contenuto invalidato dalla cache, a meno che il server di rendering non restituisca una risposta corretta. Una risposta 5xx da AEM o un timeout di connessione causa la distribuzione da parte del dispatcher del contenuto obsoleto e la risposta con uno stato HTTP 111 (revoca non riuscita).

### Memorizzazione nella cache quando viene utilizzata l'autenticazione {#caching-when-authentication-is-used}

La `/allowAuthorized` proprietà controlla se le richieste contenenti una delle seguenti informazioni di autenticazione sono memorizzate nella cache:

* The `authorization` header.
* Un cookie denominato `authorization`.
* Un cookie denominato `login-token`.

Per impostazione predefinita, le richieste che includono queste informazioni di autenticazione non vengono memorizzate nella cache perché l'autenticazione non viene eseguita quando un documento memorizzato nella cache viene restituito al client. Questa configurazione impedisce al dispatcher di distribuire i documenti memorizzati nella cache agli utenti che non dispongono dei diritti necessari.

Tuttavia, se i vostri requisiti consentono il caching dei documenti autenticati, impostate /allowAuthorized su uno:

`/allowAuthorized "1"`

>[!NOTE]
>
>Per abilitare la gestione delle sessioni (utilizzando la `/sessionmanagement` proprietà), la `/allowAuthorized` proprietà deve essere impostata su `"0"`.

### Specifica dei documenti da memorizzare nella cache {#specifying-the-documents-to-cache}

La `/rules` proprietà controlla quali documenti vengono memorizzati nella cache in base al percorso del documento. Indipendentemente dalla proprietà /rules, il dispatcher non memorizza mai nella cache un documento nelle seguenti circostanze:

* Se l’URI della richiesta contiene un punto interrogativo ("?").\
   In genere indica una pagina dinamica, ad esempio un risultato di ricerca che non deve essere memorizzato nella cache.
* Se manca l’estensione del file.\
   Il server web ha bisogno dell’estensione per determinare il tipo di documento (tipo MIME).
* Se l’intestazione di autenticazione è impostata (configurabile).
* Se l’istanza AEM risponde con le seguenti intestazioni:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>Dispatcher può memorizzare in cache i metodi GET o HEAD (per l’intestazione HTTP). Per ulteriori informazioni sul caching delle intestazioni delle risposte, consultate la sezione [Memorizzazione in cache delle intestazioni](dispatcher-configuration.md#caching-http-response-headers) di risposta HTTP.

Ogni elemento della proprietà /rules include un pattern [di tipo](#designing-patterns-for-glob-properties) di tipo:

* Il pattern di gestione viene utilizzato per corrispondere al percorso del documento.
* Il tipo indica se memorizzare nella cache i documenti che corrispondono al pattern di gestione. Il valore può essere consentito (per memorizzare il documento nella cache) o negato (per eseguire sempre il rendering del documento).

Se non disponete di pagine dinamiche (oltre a quelle già escluse dalle regole di cui sopra), potete configurare Dispatcher per memorizzare tutto nella cache. La sezione relativa alle regole si presenta come segue:

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

Per ulteriori informazioni sulle proprietà del tipo di stile, vedere [Progettazione di pattern per le proprietà](#designing-patterns-for-glob-properties)del tipo di stile.

Se alcune sezioni della pagina sono dinamiche (ad esempio un’applicazione di notizie) o all’interno di un gruppo di utenti chiuso, è possibile definire eccezioni:

>[!NOTE]
>
>I gruppi di utenti chiusi non devono essere memorizzati nella cache in quanto i diritti utente non vengono controllati per individuare le pagine memorizzate nella cache.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }   
  }
```

**Compressione**

Sui server Web Apache potete comprimere i documenti memorizzati nella cache. La compressione consente ad Apache di restituire il documento in un modulo compresso, se richiesto dal client. La compressione viene eseguita automaticamente attivando il modulo Apache `mod_deflate`, ad esempio:

```xml
AddOutputFilterByType DEFLATE text/plain
```

Il modulo è installato per impostazione predefinita con Apache 2.x.

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

### Annullamento della validità dei file in base al livello di cartella {#invalidating-files-by-folder-level}

Utilizzare la `/statfileslevel` proprietà per annullare la validità dei file memorizzati nella cache in base al percorso:

* Il dispatcher crea `.stat`i file in ciascuna cartella dalla cartella del docroot al livello specificato. La cartella docroot è di livello 0.
* I file vengono invalidati toccando il `.stat` file. L'ultima data di modifica del `.stat` file viene confrontata con l'ultima data di modifica di un documento memorizzato nella cache. Il documento viene recuperato se il `.stat` file è più recente.

* Quando un file che si trova a un determinato livello viene invalidato, **tutti** `.stat` i file dal docroot **al** livello del file invalidato o configurato `statsfilevel` (se inferiore) verranno toccati.

   * Ad esempio: se si imposta la `statfileslevel` proprietà su 6 e un file viene invalidato al livello 5, ogni `.stat` file da docroot a 5 verrà toccato. Se si continua con questo esempio, se un file viene invalidato al livello 7 allora ogni . `stat` file da docroot a 6 sarà toccato (da allora `/statfileslevel = "6"`).

Vengono interessate solo le risorse **lungo il percorso** del file invalidato. Considerate l'esempio seguente: un sito Web utilizza la struttura `/content/myWebsite/xx/.` Se impostate `statfileslevel` su 3, viene creato un `.stat`file come segue:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

Quando un file in `/content/myWebsite/xx` viene invalidato, ogni `.stat` file dal docroot verso il basso `/content/myWebsite/xx`viene toccato. Ciò vale solo per `/content/myWebsite/xx` e non per esempio `/content/myWebsite/yy` o `/content/anotherWebSite`.

>[!NOTE]
>
>È possibile impedire l'annullamento della validità inviando un'altra intestazione `CQ-Action-Scope:ResourceOnly`. Questo può essere utilizzato per cancellare risorse particolari senza invalidare altre parti della cache. Per ulteriori informazioni, consulta [questa pagina](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) e [Invalidare manualmente la cache](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) del dispatcher.

>[!NOTE]
>
>Se si specifica un valore per la `/statfileslevel` proprietà, la `/statfile` proprietà viene ignorata.

### Annullamento automatico file memorizzati nella cache {#automatically-invalidating-cached-files}

La `/invalidate` proprietà definisce i documenti che vengono automaticamente invalidati quando il contenuto viene aggiornato.

Con l'annullamento della validità automatica, il dispatcher non elimina i file memorizzati nella cache dopo un aggiornamento del contenuto, ma ne verifica la validità alla successiva richiesta. I documenti nella cache che non vengono annullati automaticamente rimarranno nella cache finché un aggiornamento del contenuto non li eliminerà in modo esplicito.

L'annullamento della validità automatica viene in genere utilizzato per le pagine HTML. Le pagine HTML spesso contengono collegamenti verso altre pagine, rendendo difficile determinare se un aggiornamento del contenuto influisce su una pagina. Per fare in modo che tutte le pagine pertinenti vengano invalidate quando viene aggiornato il contenuto, annullate automaticamente tutte le pagine HTML. La configurazione seguente invalida tutte le pagine HTML:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Per ulteriori informazioni sulle proprietà del tipo di stile, vedere [Progettazione di pattern per le proprietà](#designing-patterns-for-glob-properties)del tipo di stile.

Questa configurazione causa la seguente attività quando /content/geometrixx/en è attivato:

* Tutti i file con pattern en.* vengono rimossi dalla cartella /content/geometrixx/.
* La cartella /content/geometrixx/en/_jcr_content viene rimossa.
* Tutti gli altri file che corrispondono alla configurazione /invalidate non vengono eliminati immediatamente. Questi file vengono eliminati quando si verifica la richiesta successiva. Nel nostro esempio /content/geometrixx.html non viene eliminato, verrà eliminato quando viene richiesto /content/geometrixx.html.

Se offrite per il download file PDF e ZIP generati automaticamente, potreste dover annullare automaticamente anche questi. Esempio di configurazione:

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

L'integrazione di AEM con Adobe Analytics fornisce i dati di configurazione in un file analytics.sitecatalyst.js nel sito Web. Il file dispatcher.any fornito con Dispatcher include la seguente regola di annullamento della validità per il file:

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### Uso di script di annullamento convalida personalizzati {#using-custom-invalidation-scripts}

La proprietà /invalidateHandler consente di definire uno script che viene chiamato per ogni richiesta di annullamento della validità ricevuta dal Dispatcher.

Viene chiamato con i seguenti argomenti:

* Punto di controllo\
   Percorso del contenuto invalidato
* Azione\
   Azione di replica (ad esempio Attiva, Disattiva)
* Ambito azione\
   Ambito dell'azione di replica (vuoto, a meno che non `CQ-Action-Scope: ResourceOnly` venga inviata un'intestazione, per ulteriori informazioni, consultate [Invalidazione delle pagine nella cache da AEM](page-invalidate.md) )

Questo può essere utilizzato per coprire una serie di casi d’uso diversi, ad esempio per invalidare altre cache specifiche dell’applicazione, o per gestire i casi in cui l’URL esternalizzato di una pagina e la sua posizione nel documento non corrispondono al percorso del contenuto.

Sotto lo script di esempio, ogni richiesta di annullamento della validità viene registrata in un file.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### script del gestore di annullamento convalida di esempio {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limitazione dei client in grado di cancellare la cache {#limiting-the-clients-that-can-flush-the-cache}

La proprietà /allowClients definisce client specifici che possono cancellare la cache. I modelli globbing vengono confrontati con il PI.

Esempio:

1. nega l'accesso a qualsiasi client
1. consente esplicitamente l'accesso al localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

Per ulteriori informazioni sulle proprietà del tipo di stile, vedere [Progettazione di pattern per le proprietà](#designing-patterns-for-glob-properties)del tipo di stile.

>[!CAUTION]
>
>È consigliabile definire i /allowClient.
>
>In caso contrario, qualsiasi client può effettuare una chiamata per cancellare la cache; se questo viene fatto ripetutamente, può avere gravi ripercussioni sulle prestazioni del sito.

### Ignorare i parametri URL {#ignoring-url-parameters}

La `ignoreUrlParams` sezione definisce quali parametri URL vengono ignorati quando si determina se una pagina viene memorizzata nella cache o distribuita dalla cache:

* Quando un URL di richiesta contiene parametri che vengono tutti ignorati, la pagina viene memorizzata nella cache.
* Quando un URL di richiesta contiene uno o più parametri che non vengono ignorati, la pagina non viene memorizzata nella cache.

Quando un parametro viene ignorato per una pagina, la pagina viene memorizzata nella cache la prima volta che viene richiesta la pagina. Le richieste successive per la pagina vengono servite nella cache, indipendentemente dal valore del parametro nella richiesta.

Per specificare quali parametri vengono ignorati, aggiungete le regole di gestione alla `ignoreUrlParams` proprietà:

* Per ignorare un parametro, create una proprietà Gestione dinamica che consenta al parametro di ignorare.
* Per evitare che la pagina venga memorizzata nella cache, create una proprietà Gestione dinamica dei tag che neghi il parametro.

L’esempio seguente fa sì che il Dispatcher ignori il parametro "q", in modo che gli URL di richiesta che includono il parametro q vengano memorizzati nella cache:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

Utilizzando il valore di esempio `ignoreUrlParams` , la seguente richiesta HTTP causa la memorizzazione nella cache della pagina perché il `q` parametro viene ignorato:

```xml
GET /mypage.html?q=5
```

Utilizzando il valore di esempio `ignoreUrlParams` , la seguente richiesta HTTP fa sì che la pagina **non** venga memorizzata nella cache perché il `p` parametro non viene ignorato:

```xml
GET /mypage.html?q=5&p=4
```

Per ulteriori informazioni sulle proprietà del tipo di stile, vedere [Progettazione di pattern per le proprietà](#designing-patterns-for-glob-properties)del tipo di stile.

### Memorizzazione in cache delle intestazioni di risposta HTTP {#caching-http-response-headers}

>[!NOTE]
>
>Questa funzione è disponibile con la versione **4.1.11** del dispatcher.

La `/headers` proprietà consente di definire i tipi di intestazione HTTP che verranno memorizzati nella cache dal dispatcher. Nella prima richiesta a una risorsa non memorizzata nella cache, tutte le intestazioni corrispondenti a uno dei valori configurati (vedere l'esempio di configurazione seguente) vengono memorizzate in un file separato, accanto al file della cache. Nelle richieste successive alla risorsa memorizzata nella cache, le intestazioni memorizzate vengono aggiunte alla risposta.

Presentato di seguito è un esempio della configurazione predefinita:

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
>Inoltre, tenere presente che i caratteri di globbing del file non sono consentiti. Per ulteriori dettagli, vedere [Progettazione di pattern per le proprietà](#designing-patterns-for-glob-properties)di Gestione dinamica dei tag.

>[!NOTE]
>
>Se dovete memorizzare e distribuire le intestazioni di risposta ETag da AEM, eseguite le seguenti operazioni:
>
>* Aggiungete il nome dell’intestazione nella `/cache/headers`sezione.
>* Aggiungere la seguente direttiva [Apache](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) nella sezione relativa al dispatcher:
>



```xml
FileETag none
```

### Autorizzazioni file cache del dispatcher {#dispatcher-cache-file-permissions}

La `mode` proprietà specifica quali autorizzazioni vengono applicate alle nuove directory e ai nuovi file nella cache. Questa impostazione è limitata dal processo `umask` di chiamata. È un numero ottale costruito dalla somma di uno o più dei seguenti valori:

* 0400 Consenti lettura da parte del proprietario.
* 0200 Consenti scrittura da parte del proprietario.
* 0100 Consente al proprietario di effettuare ricerche nelle directory.
* 0040 Consenti lettura per membri del gruppo.
* 0020 Consenti scrittura per membri del gruppo.
* 0010 Consenti ai membri del gruppo di effettuare ricerche nella directory.
* 0004 Consenti lettura da parte di altri.
* 0002 Consenti scrittura da parte di altri.
* 0001 Consente ad altri di effettuare ricerche nella directory.

Il valore predefinito è 0755, che consente al proprietario di leggere, scrivere o cercare e al gruppo e ad altri di leggere o cercare.

### Throttling .stat file touch {#throttling-stat-file-touching}

Con la `/invalidate` proprietà predefinita, ogni attivazione invalida di fatto tutti `.html` i file (quando il percorso corrisponde alla `/invalidate` sezione). Su un sito Web con traffico considerevole, più attivazioni successive aumenteranno il carico di CPU sul backend. In questo caso, è consigliabile "limitare" il `.stat` tocco dei file per mantenere il sito Web reattivo. È possibile eseguire questa operazione utilizzando la `/gracePeriod` proprietà .

La `/gracePeriod` proprietà definisce il numero di secondi di cui una risorsa non aggiornata e con annullamento automatico può ancora essere servita dalla cache dopo l'ultima attivazione. La proprietà può essere utilizzata in una configurazione in cui un batch di attivazioni in caso contrario annullerebbe ripetutamente l'intera cache. Il valore consigliato è 2 secondi.

Per ulteriori dettagli, consultare anche le `/invalidate` sezioni e `/statfileslevel`sezioni precedenti.

## Configurazione dell'annullamento della validità della cache in base al tempo - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

Se impostata, la `enableTTL` proprietà valuterà le intestazioni di risposta dal back-end, e se contengono una `Cache-Control` massima età o una `Expires` data, viene creato un file ausiliario vuoto accanto al file della cache, con l'ora di modifica uguale alla data di scadenza. Quando il file memorizzato nella cache viene richiesto oltre l'ora di modifica, viene automaticamente richiesto nuovamente dal backend.

È possibile abilitare la funzione aggiungendo questa riga al `dispatcher.any` file:

```xml
/enableTTL "1"
```

>[!NOTE]
>
>Questa funzione è disponibile con la versione **4.1.11** del dispatcher.

## Configurazione del bilanciamento del carico - /statistics {#configuring-load-balancing-statistics}

La `/statistics` sezione definisce le categorie di file per i quali Dispatcher rileva la reattività di ciascun rendering. Il dispatcher utilizza i punteggi per determinare quale rendering inviare una richiesta.

Per ogni categoria creata viene definito un pattern di tipo Gap. Dispatcher confronta l'URI del contenuto richiesto con i seguenti pattern per determinare la categoria del contenuto richiesto:

* L'ordine delle categorie determina l'ordine in cui vengono confrontate con l'URI.
* Il primo pattern di categoria che corrisponde all’URI è la categoria del file. Non vengono valutati altri pattern di categoria.

Il dispatcher supporta un massimo di 8 categorie di statistiche. Se definite più di 8 categorie, vengono utilizzate solo le prime 8.

**Rendering selezione**

Ogni volta che il dispatcher richiede una pagina di cui è stato effettuato il rendering, utilizza il seguente algoritmo per selezionare il rendering:

1. Se la richiesta contiene il nome di rendering in un `renderid` cookie, il dispatcher utilizzerà tale rendering.
1. Se la richiesta non include `renderid` cookie, Dispatcher confronta le statistiche di rendering:

   1. Il dispatcher determina la categoria dell'URI della richiesta.
   1. Il dispatcher determina quale rendering ha il punteggio di risposta più basso per quella categoria e lo seleziona.

1. Se non è ancora selezionato alcun rendering, usate il primo rendering nell’elenco.

La valutazione per la categoria di un rendering è basata sui tempi di risposta precedenti, nonché sulle connessioni non riuscite e di successo precedenti tentate dal dispatcher. Per ogni tentativo, il punteggio per la categoria dell'URI richiesto viene aggiornato.

>[!NOTE]
>
>Se non si utilizza il bilanciamento del carico, è possibile omettere questa sezione.

### Definizione delle categorie di statistiche {#defining-statistics-categories}

Definire una categoria per ciascun tipo di documento per il quale si desidera mantenere le statistiche per la selezione del rendering. La sezione /statistics contiene una sezione /category. Per definire una categoria, aggiungere una riga sotto la sezione /category con il seguente formato:

`/name { /glob "pattern"}`

La categoria `name` deve essere univoca per l'azienda. La `pattern` sezione è descritta nella sezione [Modelli di progettazione per le proprietà](#designing-patterns-for-glob-properties) di Gestione dinamica dei tag.

Per determinare la categoria di un URI, Dispatcher confronta l'URI con ciascun pattern di categoria fino a trovare una corrispondenza. Il dispatcher inizia con la prima categoria nell'elenco e continua in ordine. Pertanto, inserite prima le categorie con pattern più specifici.

Ad esempio, Dispatcher il file dispatcher predefinito.any definisce una categoria HTML e un'altra. La categoria HTML è più specifica e quindi viene visualizzata per prima:

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

L’esempio seguente include anche una categoria per le pagine di ricerca:

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

### Riflesso dell'indisponibilità del server nelle statistiche del dispatcher {#reflecting-server-unavailability-in-dispatcher-statistics}

La `/unavailablePenalty` proprietà imposta il tempo (in decimi di secondo) applicato alle statistiche di rendering quando una connessione al rendering non riesce. Il dispatcher aggiunge l'ora alla categoria di statistiche che corrisponde all'URI richiesto.

Ad esempio, la sanzione viene applicata quando non è possibile stabilire la connessione TCP/IP alla porta/nome host designata, perché AEM non è in esecuzione (e non è in ascolto) o a causa di un problema di rete.

La `/unavailablePenalty` proprietà è un elemento secondario diretto della `/farm` sezione (un elemento di pari livello della `/statistics` sezione).

Se non esiste alcuna `/unavailablePenalty` proprietà, viene utilizzato il valore "1".

```xml
/unavailablePenalty "1"
```

## Identificazione di una cartella di connessione fissa - /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

La `/stickyConnectionsFor` proprietà definisce una cartella contenente documenti fissi; l’accesso verrà eseguito utilizzando l’URL. Il dispatcher invia tutte le richieste, da un singolo utente, presenti in questa cartella alla stessa istanza di rendering. Le connessioni permanenti garantiscono la presenza e la coerenza dei dati della sessione per tutti i documenti. Questo meccanismo utilizza il `renderid` cookie.

L'esempio seguente definisce una connessione fissa alla cartella /products:

```xml
/stickyConnectionsFor "/products"
```

Quando una pagina è composta da contenuto proveniente da più nodi di contenuto, includete la `/paths` proprietà che elenca i percorsi del contenuto. Ad esempio, una pagina contiene il contenuto proveniente da `/content/image`, `/content/video`e `/var/files/pdfs`. La seguente configurazione abilita connessioni fisse per tutto il contenuto della pagina:

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

Quando sono attivate le connessioni di tipo "sticky", il modulo del dispatcher imposta il `renderid` cookie. Questo cookie non ha la `httponly` bandiera, che dovrebbe essere aggiunta per migliorare la sicurezza. È possibile eseguire questa operazione impostando la `httpOnly` proprietà nel `/stickyConnections` nodo di un file di `dispatcher.any` configurazione. Il valore della proprietà (0 o 1) definisce se al `renderid` cookie è aggiunto l' `HttpOnly` attributo. Il valore predefinito è 0, ovvero l'attributo non verrà aggiunto.

Per ulteriori informazioni sul `httponly` flag, leggete [questa pagina](https://www.owasp.org/index.php/HttpOnly).

### secure {#secure}

Quando sono attivate le connessioni di tipo "sticky", il modulo del dispatcher imposta il `renderid` cookie. Questo cookie non ha il flag **sicuro** , che deve essere aggiunto per migliorare la sicurezza. È possibile eseguire questa operazione impostando la `secure` proprietà nel `/stickyConnections` nodo di un file di `dispatcher.any` configurazione. Il valore della proprietà (0 o 1) definisce se al `renderid` cookie è aggiunto l' `secure` attributo. Il valore predefinito è 0, ovvero l’attributo verrà aggiunto **se** la richiesta in arrivo è sicura. Se il valore è impostato su 1, il flag secure verrà aggiunto indipendentemente dal fatto che la richiesta in arrivo sia protetta o meno.

## Gestione degli errori di connessione del rendering {#handling-render-connection-errors}

Configura il comportamento del dispatcher quando il server di rendering restituisce un errore 500 o non è disponibile.

### Specifica di una pagina di verifica dello stato {#specifying-a-health-check-page}

Utilizzare la `/health_check` proprietà per specificare un URL controllato quando si verifica un codice di stato 500. Se questa pagina restituisce anche un codice di stato 500, l’istanza viene considerata non disponibile e al rendering viene applicata una penale ( `/unavailablePenalty`) configurabile prima di riprovare.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifica del ritardo del tentativo di pagina {#specifying-the-page-retry-delay}

La proprietà / `retryDelay` imposta il tempo (in secondi) che il dispatcher attende tra i cicli di tentativi di connessione con i rendering della farm. Per ogni arrotondamento, il numero massimo di tentativi di dispatcher di creare una connessione a un rendering corrisponde al numero di rendering nella farm.

Il dispatcher utilizza un valore di `"1"` se non `/retryDelay` è definito in modo esplicito. Il valore predefinito è appropriato nella maggior parte dei casi.

```xml
/retryDelay "1"
```

### Configurazione del numero di tentativi {#configuring-the-number-of-retries}

La `/numberOfRetries` proprietà imposta il numero massimo di cicli di tentativi di connessione eseguiti da Dispatcher con i rendering. Se Dispatcher non riesce a collegarsi a un rendering dopo questo numero di tentativi, Dispatcher restituisce una risposta non riuscita.

Per ogni arrotondamento, il numero massimo di tentativi di dispatcher di creare una connessione a un rendering corrisponde al numero di rendering nella farm. Pertanto, il numero massimo di tentativi di connessione da parte del dispatcher è ( `/numberOfRetries`) x (il numero di rendering).

Se il valore non è esplicitamente definito, il valore predefinito è **5**.

```xml
/numberOfRetries "5"
```

### Utilizzo del meccanismo di failover {#using-the-failover-mechanism}

Abilitate il meccanismo di failover nella farm del dispatcher per inviare nuovamente le richieste a diversi rendering quando la richiesta originale non riesce. Quando il failover è abilitato, il dispatcher ha il seguente comportamento:

* Quando una richiesta a un rendering restituisce lo stato HTTP 503 (NON DISPONIBILE), il dispatcher invia la richiesta a un rendering diverso.
* Quando una richiesta a un rendering restituisce lo stato HTTP 50x (diverso da 503), il dispatcher invia una richiesta per la pagina configurata per la `health_check` proprietà.

   * Se il controllo dello stato restituisce 500 (INTERNAL_SERVER_ERROR), il dispatcher invia la richiesta originale a un rendering diverso.
   * Se il controllo healtch restituisce lo stato HTTP 200, Dispatcher restituisce l'errore HTTP 500 iniziale al client.

Per abilitare il failover, aggiungi la seguente riga alla farm (o al sito Web):

```xml
/failover "1" 
```

>[!NOTE]
>
>Per riprovare le richieste HTTP contenenti un corpo, Dispatcher invia un'intestazione di `Expect: 100-continue` richiesta al rendering prima di spooling del contenuto effettivo. CQ 5.5 con CQSE risponde immediatamente con 100 (CONTINUA) o con un codice di errore. Anche altri contenitori servlet dovrebbero supportare questo.

## Ignorare gli errori di interruzione - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Questa opzione in genere non è necessaria. È sufficiente utilizzarlo per visualizzare i seguenti messaggi di registro:
>
>`Error while reading response: Interrupted system call`

Qualsiasi chiamata di sistema orientata al file system può essere interrotta `EINTR` se l'oggetto della chiamata di sistema si trova in un sistema remoto a cui si accede tramite NFS. Se queste chiamate di sistema possono essere interrotte o interrotte, si basa su come il file system sottostante è stato montato sul computer locale.

Utilizzate il parametro /ignoreEINTR se la vostra istanza dispone di tale configurazione e il registro contiene il seguente messaggio:

`Error while reading response: Interrupted system call`

Internamente, il dispatcher legge la risposta dal server remoto (ad es. AEM) utilizzando un loop che può essere rappresentato come:

`while (response not finished) {  
read more data  
}`

Tali messaggi possono essere generati quando `EINTR` si verifica nella sezione " `read more data`" e sono causati dalla ricezione di un segnale prima della ricezione dei dati.

Per ignorare tali interruzioni potete aggiungere il seguente parametro a `dispatcher.any` (prima `/farms`):

`/ignoreEINTR "1"`

Se si imposta `/ignoreEINTR` `"1"` questo parametro, Dispatcher continuerà a tentare di leggere i dati fino alla lettura della risposta completa. Il valore predefinito è 0 e disattiva l’opzione.

## Progettazione di pattern per le proprietà di Gestione dinamica dei tag {#designing-patterns-for-glob-properties}

Diverse sezioni del file di configurazione del dispatcher utilizzano `glob` le proprietà come criteri di selezione per le richieste dei client. I valori delle proprietà di controllo dell'integrità sono pattern che il dispatcher confronta con un aspetto della richiesta, ad esempio il percorso della risorsa richiesta o l'indirizzo IP del client. Ad esempio, gli elementi nella `/filter` sezione utilizzano pattern di tipo Gap per identificare i percorsi delle pagine su cui il Dispatcher agisce o che rifiuta.

I valori di GLOW possono includere caratteri jolly e caratteri alfanumerici per definire il pattern.

| Carattere jolly | Descrizione | Esempi |
|--- |--- |--- |
| `*` | Corrisponde a zero o più istanze contigue di qualsiasi carattere nella stringa. Il carattere finale della corrispondenza è determinato dalle seguenti situazioni: <br/>Un carattere nella stringa corrisponde al carattere successivo nel pattern e il carattere del pattern ha le seguenti caratteristiche:<br/><ul><li>Not a *</li><li>Non è un ?</li><li>Un carattere letterale (incluso uno spazio) o una classe di caratteri.</li><li>Viene raggiunta la fine del pattern.</li></ul>All'interno di una classe di caratteri, il carattere viene interpretato letteralmente. | `*/geo*` Corrisponde a qualsiasi pagina sotto il `/content/geometrixx` nodo e il `/content/geometrixx-outdoors` nodo. Le seguenti richieste HTTP corrispondono al pattern di gestione: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` Corrisponde <br/>a qualsiasi pagina sotto il `/content/geometrixx-outdoors` nodo. Ad esempio, la seguente richiesta HTTP corrisponde al pattern di gestione: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Corrisponde a qualsiasi singolo carattere. Utilizzare classi di caratteri esterne. All'interno di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*outdoors/??/*`<br/> Corrisponde alle pagine per qualsiasi lingua del sito geometrixx-outdoors. Ad esempio, la seguente richiesta HTTP corrisponde al pattern di gestione: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>La richiesta seguente non corrisponde al pattern di tipo Gap: <br/><ul><li>"GET /content/geometrixx-outdoors/en.html"</li></ul> |
| `[ and ]` | Richiama l'inizio e la fine di una classe di caratteri. Le classi di caratteri possono includere uno o più intervalli di caratteri e caratteri singoli.<br/>Una corrispondenza si verifica se il carattere di destinazione corrisponde a uno qualsiasi dei caratteri della classe di caratteri, o all'interno di un intervallo definito.<br/>Se la parentesi di chiusura non è inclusa, il pattern non produce alcuna corrispondenza. | `*[o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` Corrisponde <br/>alle seguenti richieste HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Indica un intervallo di caratteri. Da utilizzare nelle classi di caratteri.  Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[m-p]men.html*` Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Nega la classe di caratteri o di caratteri che segue. Utilizzare solo per negare i caratteri e gli intervalli di caratteri all'interno delle classi di caratteri. Equivalente al `^ wildcard`. <br/>Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[!o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` o `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Nega il carattere o l'intervallo di caratteri che segue. Utilizzare per negare solo caratteri e intervalli di caratteri all'interno delle classi di caratteri. Equivalente al carattere `!` jolly. <br/>Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | Si applicano gli esempi del carattere `!` jolly, sostituendo i `!` caratteri nei pattern di esempio con `^` caratteri. |


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

Nella configurazione del server Web, potete impostare:

* Posizione del file di registro del dispatcher.
* Livello di registro.

Per ulteriori informazioni, consultare la documentazione del server Web e il file Leggimi dell’istanza del Dispatcher.

**Registri Apache ruotati/Piped**

Se utilizzate un server Web **Apache** potete utilizzare la funzionalità standard per i registri ruotati e/o con tubazioni. Ad esempio, utilizzando i registri con tubazioni:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Questo verrà ruotato automaticamente:

* il file di registro del dispatcher; con una marca temporale nell'estensione (logs/dispatcher.log%Y%m%d).
* su base settimanale (60 x 60 x 24 x 7 = 604800 secondi).

Consulta la documentazione del server web Apache su Log Rotation e Piped Logs; ad esempio [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Al momento dell'installazione, il livello di registro predefinito è elevato (ovvero livello 3 = Debug), in modo che il dispatcher registri tutti gli errori e gli avvisi. Questo è molto utile nelle fasi iniziali.
>
>Tuttavia, questo richiede risorse aggiuntive, pertanto, quando il Dispatcher funziona correttamente *in base alle esigenze*, è possibile (dovrebbe) ridurre il livello di registro.

### Registrazione traccia {#trace-logging}

Tra gli altri miglioramenti apportati al dispatcher, la versione 4.2.0 introduce anche la funzione di registrazione delle tracce.

Si tratta di un livello superiore alla registrazione di debug, che mostra informazioni aggiuntive nei registri. Aggiunge la registrazione per:

* I valori delle intestazioni inoltrate;
* Regola applicata per una determinata azione.

È possibile abilitare la funzione di registrazione traccia impostando il livello di registro `4` nel server Web.

Di seguito è riportato un esempio di registri con traccia abilitata:

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

## Conferma del funzionamento di base {#confirming-basic-operation}

Per confermare il funzionamento e l’interazione di base tra il server Web, il dispatcher e l’istanza di AEM, procedi come segue:

1. Impostare `loglevel` su `3`.

1. Avviare il server Web; viene avviato anche il Dispatcher.
1. Avviate l’istanza AEM.
1. Controllate i file di registro e di errore del server Web e del dispatcher.\
   A seconda del server Web, dovrebbero essere visualizzati messaggi quali:\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   e:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Navigare sul sito Web tramite il server Web. Verificate che il contenuto venga visualizzato come richiesto.\
   Ad esempio, in un’installazione locale in cui AEM viene eseguito sulla porta `4502` e sul server Web che accede `80` alla console Siti Web utilizzando entrambi:\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`I risultati devono essere identici. Confermate l'accesso ad altre pagine con lo stesso meccanismo.

1. Verificate che la directory della cache sia stata compilata.
1. Attivate una pagina per verificare che la cache venga scaricata correttamente.
1. Se tutto funziona correttamente è possibile ridurre il `loglevel` a `0`.

## Utilizzo di più istanze di Dispatcher {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più istanze di Dispatcher. Ad esempio, puoi utilizzare:

* un’istanza di Dispatcher per pubblicare un sito web nella Intranet;
* una seconda istanza di Dispatcher, con un indirizzo diverso e impostazioni di sicurezza diverse, per pubblicare lo stesso contenuto in Internet.

In tal caso, assicurati che ogni richiesta venga gestita tramite un’unica istanza di Dispatcher. Un’istanza di Dispatcher non gestisce le richieste provenienti da un’altra istanza di Dispatcher. Assicurati pertanto che entrambe le istanze di Dispatcher accedano direttamente al sito web AEM.

## Debug {#debugging}

Quando si aggiunge l'intestazione `X-Dispatcher-Info` a una richiesta, il dispatcher risponde se la destinazione è stata memorizzata nella cache, è stata restituita dalla cache o non è possibile memorizzarla nella cache. L'intestazione della risposta `X-Cache-Info` contiene queste informazioni in un modulo leggibile. È possibile utilizzare queste intestazioni di risposta per risolvere i problemi relativi alle risposte memorizzate nella cache del dispatcher.

Questa funzionalità non è abilitata per impostazione predefinita, pertanto per includere l'intestazione della risposta, `X-Cache-Info` la farm deve contenere la seguente voce:

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

Inoltre, l' `X-Dispatcher-Info` intestazione non ha bisogno di un valore, ma se utilizzate `curl` per il test dovete fornire un valore per inviare l'intestazione, ad esempio:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/we-retail/us/en.html
```

Di seguito è riportato un elenco contenente le intestazioni di risposta che `X-Dispatcher-Info` restituiranno:

* **cache**\
   Il file di destinazione è contenuto nella cache e il dispatcher ha stabilito che è valido per distribuirlo.
* **caching**\
   Il file di destinazione non è contenuto nella cache e il dispatcher ha stabilito che è valido per memorizzare l'output nella cache e distribuirlo.
* **caching: Il file stat è più recente** Il file di destinazione è contenuto nella cache, tuttavia, viene invalidato da un file di stato più recente. Il dispatcher eliminerà il file di destinazione, lo ricreerà dall'output e lo invierà.
* **non memorizzabile nella cache: nessun elemento principale** del documento La configurazione della farm non contiene un elemento radice del documento (elemento di configurazione `cache.docroot`).
* **non memorizzabile nella cache: percorso del file cache troppo lungo**\
   Il file di destinazione, ovvero la concatenazione della radice del documento e del file URL, supera il nome file più lungo possibile sul sistema.
* **non memorizzabile nella cache: percorso del file temporaneo troppo lungo**\
   Il modello di nome file temporaneo supera il nome file più lungo possibile sul sistema. Il dispatcher crea prima un file temporaneo, prima di creare o sovrascrivere effettivamente il file memorizzato nella cache. Il nome del file temporaneo è il nome del file di destinazione con i caratteri `_YYYYXXXXXX` aggiunti, in cui il nome `Y` e `X` verrà sostituito per creare un nome univoco.
* **non memorizzabile nella cache: l'URL della richiesta non ha estensione**\
   L'URL della richiesta non ha alcuna estensione oppure esiste un percorso dopo l'estensione del file, ad esempio: `/test.html/a/path`.
* **non memorizzabile nella cache: La richiesta non era un metodo GET o HEAD** Il metodo HTTP non è né un metodo GET né un metodo HEAD. Il dispatcher presume che l'output conterrà dati dinamici che non dovrebbero essere memorizzati nella cache.
* **non memorizzabile nella cache: la richiesta conteneva una stringa di query**\
   La richiesta conteneva una stringa di query. Il dispatcher presuppone che l'output dipenda dalla stringa di query specificata e pertanto non memorizza nella cache.
* **non memorizzabile nella cache: session manager non autenticato**\
   La cache della farm è gestita da un manager sessione (la configurazione contiene un `sessionmanagement` nodo) e la richiesta non conteneva le informazioni di autenticazione appropriate.
* **non memorizzabile nella cache: la richiesta contiene l'autorizzazione**\
   La farm non può memorizzare nella cache l'output ( `allowAuthorized 0`) e la richiesta contiene informazioni di autenticazione.
* **non memorizzabile nella cache: target è una directory**\
   Il file di destinazione è una directory. Ciò potrebbe indicare un errore concettuale, in cui un URL e alcuni URL secondari contengono entrambi output memorizzabili nella cache, ad esempio se una richiesta `/test.html/a/file.ext` viene prima e contiene output memorizzabile nella cache, il dispatcher non sarà in grado di memorizzare nella cache l'output di una richiesta successiva a `/test.html`.
* **non memorizzabile nella cache: l'URL della richiesta ha una barra finale**\
   L’URL della richiesta dispone di una barra finale.
* **non memorizzabile nella cache: URL richiesta non presente nelle regole della cache**\
   Le regole della cache della farm negano esplicitamente la memorizzazione nella cache dell'output di alcuni URL di richieste.
* **non memorizzabile nella cache: accesso negato al controllo dell'autorizzazione**\
   Il controllo delle autorizzazioni della farm ha negato l'accesso al file memorizzato nella cache.
* **non memorizzabile nella cache: session non valida** La cache della farm è gestita da un gestore di sessioni (la configurazione contiene un `sessionmanagement` nodo) e la sessione dell'utente non è o non è più valida.
* **non memorizzabile nella cache: la risposta contiene`no_cache `** Il server remoto ha restituito un' `Dispatcher: no_cache` intestazione che impediva al dispatcher di memorizzare nella cache l'output.
* **non memorizzabile nella cache: la lunghezza del contenuto della risposta è zero** La lunghezza del contenuto della risposta è zero; il dispatcher non creerà un file di lunghezza zero.
