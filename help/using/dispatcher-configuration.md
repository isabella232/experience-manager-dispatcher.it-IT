---
title: Configurazione di Dispatcher
description: Scopri come configurare Dispatcher.
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: deb232be3c4c5e3d11d13cbccb282409d90b93bb
workflow-type: tm+mt
source-wordcount: '8528'
ht-degree: 100%

---

# Configurazione di Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

Le sezioni seguenti descrivono come configurare vari aspetti di Dispatcher.

## Supporto per IPv4 e IPv6 {#support-for-ipv-and-ipv}

Tutti gli elementi di AEM e Dispatcher possono essere installati nelle reti IPv4 e IPv6. Vedi [IPV4 e IPV6](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=it#ipv-and-ipv).

## File di configurazione di Dispatcher {#dispatcher-configuration-files}

Per impostazione predefinita, la configurazione di Dispatcher viene memorizzata nel file di testo `dispatcher.any`, anche se è possibile modificare il nome e la posizione del file durante l’installazione.

Il file di configurazione contiene una serie di proprietà con valore singolo o più valori che controllano il comportamento di Dispatcher:

* I nomi delle proprietà hanno una barra (`/`) come prefisso.
* Nella proprietà con più valori gli elementi figlio sono racchiusi tra parentesi graffe (`{ }`).

Esempio di configurazione strutturata come segue:

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

Puoi includere altri file che contribuiscono alla configurazione:

* Se il file di configurazione è molto grande, puoi suddividerlo in file più piccoli (più facili da gestire) e includerli.
* Per includere i file generati automaticamente.

Ad esempio, per includere il file myFarm.any nella configurazione /farms, utilizza il seguente codice:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Utilizza l’asterisco (`*`) come carattere jolly per specificare un intervallo di file da includere.

Ad esempio, se i file da `farm_1.any` a `farm_5.any` contengono la configurazione delle farm da uno a cinque, puoi includerli come segue:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Utilizzo delle variabili di ambiente {#using-environment-variables}

Puoi utilizzare le variabili di ambiente nelle proprietà con valori stringa nel file dispatcher.any, invece di utilizzare valori a codifica fissa (hard-coding). Per includere il valore di una variabile di ambiente, utilizza il formato `${variable_name}`.

Ad esempio, se il file dispatcher.any si trova nella stessa directory della cache, è possibile utilizzare il seguente valore per la proprietà [docroot](#specifying-the-cache-directory):

```xml
/docroot "${PWD}/cache"
```

Altro esempio: se crei una variabile di ambiente denominata `PUBLISH_IP` che memorizza il nome host dell’istanza AEM Publish, puoi utilizzare la seguente configurazione della proprietà [/renders](#defining-page-renderers-renders):

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Denominazione dell’istanza di Dispatcher {#naming-the-dispatcher-instance-name}

Utilizza la proprietà `/name` per specificare un nome univoco per identificare l’istanza di Dispatcher. La proprietà `/name` è una proprietà di alto livello nella struttura della configurazione.

## Definizione delle farm {#defining-farms-farms}

La proprietà `/farms` definisce uno o più set di comportamenti di Dispatcher e ciascun set è associato a siti web o URL diversi. La proprietà `/farms` può includere una o più farm:

* Utilizza una singola farm se vuoi che Dispatcher gestisca allo stesso modo tutte le pagine o i siti web.
* Crea più farm quando diverse aree del sito web o diversi siti web richiedono un comportamento diverso da Dispatcher.

La proprietà `/farms` è una proprietà di alto livello nella struttura della configurazione. Per definire una farm, aggiungi una proprietà figlio alla proprietà `/farms`. Utilizza un nome di proprietà che identifichi in modo univoco la farm all’interno dell’istanza di Dispatcher.

La proprietà `/farmname` ha più valori e contiene altre proprietà che definiscono il comportamento di Dispatcher:

* Gli URL delle pagine a cui si applica la farm.
* Uno o più URL di servizio (in genere, istanze AEM Publish) da utilizzare per il rendering dei documenti.
* Le statistiche da utilizzare per il bilanciamento del carico tra più renderer di documenti.
* Diversi altri comportamenti, ad esempio quali file memorizzare in cache e dove.

Il valore può includere qualsiasi carattere alfanumerico (a-z, 0-9). L’esempio che segue mostra la definizione dello scheletro per due farm denominate `/daycom` e `/docsdaycom`:

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
>Se utilizzi più di una farm di rendering, l’elenco viene valutato dal basso verso l’alto. Ciò è particolarmente importante quando definisci [host virtuali](#identifying-virtual-hosts-virtualhosts) per i siti web.

Ogni proprietà farm può contenere le seguenti proprietà figlio:

| Nome proprietà | Descrizione |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | Home page predefinita (opzionale)(solo IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | Le intestazioni della richiesta HTTP del client da inviare. |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | Host virtuali di questa farm. |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | Supporto per la gestione e l’autenticazione delle sessioni. |
| [/renders](#defining-page-renderers-renders) | I server che forniscono le pagine sottoposte a rendering (in genere istanze AEM Publish). |
| [/filter](#configuring-access-to-content-filter) | Definisce gli URL ai quali Dispatcher consente l’accesso. |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | Configura l’accesso agli URL personalizzati. |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | Supporto per l’inoltro delle richieste di distribuzione di contenuto (syndication). |
| [/cache](#configuring-the-dispatcher-cache-cache) | Configura il comportamento del caching. |
| [/statistics](#configuring-load-balancing-statistics) | Definizione di categorie statistiche per i calcoli di bilanciamento del carico. |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | Cartella contenente documenti permanenti. |
| [/health_check](#specifying-a-health-check-page) | URL da utilizzare per determinare la disponibilità del server. |
| [/retryDelay](#specifying-the-page-retry-delay) | Ritardo prima di ritentare una connessione non riuscita. |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | Penalità che incidono sulle statistiche relative ai calcoli di bilanciamento del carico. |
| [/failover](#using-the-failover-mechanism) | Invia di nuovo le richieste a diversi rendering quando la richiesta originale non riesce. |
| [/auth_checker](permissions-cache.md) | Per il caching sensibile alle autorizzazioni, consulta [Caching di contenuto protetto](permissions-cache.md). |

## Specifica di una pagina predefinita (solo IIS): /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>Il parametro `/homepage` (solo IIS) non funziona più. In Alternativa, utilizza [URL Rewrite Module per Microsoft IIS](https://docs.microsoft.com/it-it/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Se utilizzi Apache, utilizza il modulo `mod_rewrite`. Per informazioni su `mod_rewrite`, vedi la documentazione del sito web di Apache (ad esempio, [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). Quando si utilizza `mod_rewrite`, è consigliabile utilizzare il flag **[‘passthrough|PT’ (passare all’handler successivo)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** per forzare il motore di riscrittura a impostare il campo `uri` della struttura interna `request_rec` sul valore del campo `filename`.

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

La proprietà `/clientheaders` definisce un elenco di intestazioni HTTP che Dispatcher trasmette dalla richiesta HTTP del client al renderer (istanza di AEM).

Per impostazione predefinita, Dispatcher inoltra le intestazioni HTTP standard all’istanza di AEM. In alcuni casi, potrebbe essere utile inoltrare intestazioni aggiuntive o rimuovere intestazioni specifiche:

* Aggiungi le intestazioni, ad esempio intestazioni personalizzate, che l’istanza di AEM si aspetta di trovare nella richiesta HTTP.
* Rimuovi le intestazioni, ad esempio intestazioni di autenticazione, che sono rilevanti solo per il server web.

Se personalizzi il set di intestazioni da trasmettere, devi specificare un elenco completo di intestazioni, incluse quelle che sono normalmente già in uso per impostazione predefinita.

Ad esempio, un’istanza di Dispatcher che gestisce le richieste di attivazione delle pagine per le istanze Publish richiede l’intestazione `PATH` nella sezione `/clientheaders`. L’intestazione `PATH` abilita la comunicazione tra l’agente di replica e Dispatcher.

Il codice che segue è un esempio di configurazione per `/clientheaders`:

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

## Identificazione di host virtuali {#identifying-virtual-hosts-virtualhosts}

La proprietà `/virtualhosts` definisce un elenco di tutte le combinazioni nome host/URI accettate da Dispatcher per questa farm. È possibile utilizzare il carattere asterisco (`*`) come carattere jolly. I valori della proprietà / `virtualhosts` utilizzano il seguente formato:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Facoltativo) `https://` oppure `https://.`
* `host`: nome o indirizzo IP del computer host e, se necessario, numero di porta. (Vedi [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`: (Facoltativo) percorso delle risorse.

Nell’esempio che segue la configurazione gestisce le richieste per i domini .com e .ch di myCompany e per tutti i domini di mySubDivision:

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

La configurazione che segue gestisce *tutte* le richieste:

```xml
   /virtualhosts
    {
    "*"
    }
```

### Risoluzione dell’host virtuale {#resolving-the-virtual-host}

Quando Dispatcher riceve una richiesta HTTP o HTTPS, trova il valore dell’host virtuale che negli corrisponde alle intestazioni `host,` `uri` e `scheme` della richiesta. Dispatcher valuta i valori delle proprietà `virtualhosts` nel seguente ordine:

* Dispatcher inizia dalla farm più bassa e procede verso l’alto nel file dispatcher.any.
* Per viascuna farm, Dispatcher inizia con il valore più in alto nella proprietà `virtualhosts` e procede verso il basso nell’elenco dei valori.

Dispatcher trova il valore dell’host virtuale più corrispondente nel modo seguente:

* Viene utilizzato il primo host virtuale rilevato che corrisponde a tutte e tre le proprietà `host`, `scheme` e `uri` della richiesta.
* Se nessun valore della proprietà `virtualhosts` contiene parti delle proprietà`scheme` e `uri` corrispondenti alle proprietà `scheme` e `uri` della richiesta, viene utilizzato il primo host virtuale rilevato che corrisponde alla proprietà `host` della richiesta.
* Se nessun valore della proprietà `virtualhosts` ha una parte host corrispondente all’host della richiesta, viene utilizzato l’host virtuale più in alto della farm più in alto.

Pertanto, devi posizionare l’host virtuale predefinito nella parte superiore della proprietà `virtualhosts` nella farm più in alto del file `dispatcher.any`.

### Esempio di risoluzione dell’host virtuale {#example-virtual-host-resolution}

L’esempio che segue è un frammento di codice preso da un file `dispatcher.any` che definisce due farm di Dispatcher e ciascuna farm definisce una proprietà `virtualhosts`.

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

Utilizzando questo esempio, la tabella che segue mostra gli host virtuali risolti per le richieste HTTP specificate:

| URL richiesta | Host virtuale risolto |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## Abilitazione di sessioni sicure: /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>Per abilitare questa funzione, la proprietà `/allowAuthorized` **deve** essere impostata su `"0"` nella sezione `/cache`.

Crea una sessione protetta per l’accesso alla farm di rendering, in modo che gli utenti debbano effettuare il login per accedere a qualsiasi pagina della farm. Una volta effettuato il login, gli utenti possono accedere alle pagine della farm. Per informazioni sull’utilizzo di questa funzione con i gruppi utenti chiusi (CUG), vedi [Creazione di un gruppo utenti chiuso](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=it#creating-the-user-group-to-be-used). Inoltre, vedi [Elenco di controllo della sicurezza](/help/using/security-checklist.md) di Dispatcher prima di andare “live”.

La proprietà `/sessionmanagement` è una sottoproprietà di `/farms`.

>[!CAUTION]
>
>Se le sezioni del sito web utilizzano requisiti di accesso diversi, è necessario definire più farm.

**/sessionmanagement** ha diversi sottoparametri:

**/directory** (obbligatorio)

La directory in cui sono memorizzate le informazioni della sessione. Se la directory non esiste, viene creata.

>[!CAUTION]
>
> Durante la configurazione del sottoparametro della directory **non** puntare alla cartella principale (`/directory "/"`), in quanto ciò può causare gravi problemi. È sempre necessario specificare il percorso della cartella in cui sono memorizzate le informazioni della sessione. Esempio:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (facoltativo)

Codifica delle informazioni della sessione. Utilizza `md5` per la crittografia, utilizzando l’algoritmo md5 oppure `hex` per la codifica esadecimale. Se crittografi i dati della sessione, un utente con accesso al file system non può leggere il contenuto della sessione. Il valore predefinito è `md5`.

**/header** (facoltativo)

Nome dell’intestazione HTTP o del cookie che memorizza le informazioni di autorizzazione. Se memorizzi le informazioni nell’intestazione http, utilizza `HTTP:<header-name>`. Per memorizzare le informazioni in un cookie, utilizza `Cookie:<header-name>`. Se non specifichi alcun valore, viene utilizzato `HTTP:authorization`.

**/timeout** (facoltativo)

Il numero di secondi prima che la sessione vada in timeout dopo l’ultimo utilizzo. Se non specificato, viene utilizzato `"800"`, quindi sessione va in timeout poco più di 13 minuti dopo l’ultima richiesta dell’utente.

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

## Definizione dei renderer di pagine {#defining-page-renderers-renders}

La proprietà /renders definisce l’URL a cui Dispatcher invia le richieste di rendering di un documento. L’esempio di sezione `/renders` che segue identifica una singola istanza di AEM per il rendering:

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

L’esempio di sezione /renders che segue identifica un’istanza di AEM che viene eseguita sullo stesso computer di Dispatcher:

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

La seguente sezione di esempio /renders distribuisce le richieste di rendering equamente tra due istanze di AEM:

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

Specifica il timeout di connessione per l’accesso all’istanza di AEM espresso in millisecondi. Il valore predefinito è `"0"` e Dispatcher deve attendere indefinitamente.

**/receiveTimeout**

Specifica il tempo in millisecondi che una risposta può richiedere. Il valore predefinito è `"600000"` e Dispatcher deve attendere 10 minuti. L’impostazione `"0"` elimina completamente il timeout.

Se viene raggiunto il timeout durante l’analisi delle intestazioni di risposta, viene restituito lo stato HTTP 504 (Gateway non valido). Se il timeout viene raggiunto durante la lettura del corpo della risposta, Dispatcher restituisce al client la risposta incompleta, ma elimina tutti i file di cache che potrebbero essere stati scritti.

**/ipv4**

Specifica se Dispatcher utilizza la funzione `getaddrinfo` (per IPv6) o la funzione `gethostbyname` (per IPv4) per ottenere l’indirizzo IP del rendering. Se si imposta il valore 0, viene utilizzato `getaddrinfo`. Se si imposta il valore `1`, viene utilizzato `gethostbyname`. Il valore predefinito è `0`.

La funzione `getaddrinfo` restituisce un elenco di indirizzi IP. Dispatcher scorre l’elenco degli indirizzi fino a quando non stabilisce una connessione TCP/IP. Pertanto, la proprietà `ipv4` è importante quando il nome host di rendering è associato a più indirizzi IP e l’host, in risposta alla funzione `getaddrinfo`, restituisce un elenco di indirizzi IP che sono sempre nello stesso ordine. In questa situazione, utilizza la funzione `gethostbyname` in modo che l’indirizzo IP con cui Dispatcher si connette sia randomizzato.

L’ELB (Elastic Load Balancing) di Amazon è un servizio che risponde a getaddrinfo con un elenco potenzialmente identico di indirizzi IP.

**/secure**

Se per la proprietà `/secure` è impostato il valore `"1"`, Dispatcher utilizza HTTPS per comunicare con l’istanza di AEM. Per ulteriori dettagli, consulta anche [Configurazione di Dispatcher per utilizzare SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Con la versione di Dispatcher **4.1.6**, puoi configurare la proprietà `/always-resolve` come segue:

* Se impostata su `"1"`, risolverà il nome host in ogni richiesta (Dispatcher non memorizzerà mai in cache alcun indirizzo IP). Potrebbe verificarsi un leggero impatto sulle prestazioni a causa della chiamata aggiuntiva necessaria per ottenere le informazioni host per ciascuna richiesta.
* Se la proprietà non è impostata, l’indirizzo IP verrà memorizzato in cache per impostazione predefinita.

Questa proprietà può essere utilizzata anche in caso di problemi di risoluzione degli IP dinamici, come illustrato nell’esempio che segue:

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

## Configurazione dell’accesso al contenuto {#configuring-access-to-content-filter}

Utilizza la sezione `/filter` per specificare le richieste HTTP che Dispatcher può accettare. Tutte le altre richieste vengono rimandate al server web con codice di errore 404 (pagina non trovata). Se non esiste alcuna sezione `/filter`, tutte le richieste vengono accettate.

**Nota:** le richieste di [statfile](#naming-the-statfile) vengono sempre rifiutate.

>[!CAUTION]
>
>Per ulteriori considerazioni sulla limitazione dell’accesso tramite Dispatcher, vedi [Elenco di controllo della sicurezza di Dispatcher](security-checklist.md). Inoltre, leggi [Elenco di controllo della sicurezza AEM](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/security-checklist.html?lang=it#security) per ulteriori dettagli sulla sicurezza relativi all’installazione di AEM.

La sezione `/filter` consiste di una serie di regole che negano o consentono l’accesso al contenuto in base ai modelli presenti nella parte della riga della richiesta HTTP. Utilizza un elenco Consentiti per la sezione `/filter`:

* Per prima cosa, nega l’accesso a tutto.
* Quindi, consenti l’accesso al contenuto in base alle esigenze.

>[!NOTE]
>
>Si consiglia di eliminare la cache ogni volta che si effettuano modifiche nelle regole del filtro.

### Definizione di un filtro {#defining-a-filter}

Ogni elemento della sezione `/filter` include un tipo e un modello corrispondenti a un elemento specifico della riga di richiesta o all’intera riga di richiesta. Ciascun filtro può contenere i seguenti elementi:

* **Tipo**: `/type` indica se consentire o negare l’accesso delle richieste che corrispondono al modello. Il valore può essere `allow` oppure `deny`.

* **Elemento della riga di richiesta:** Include `/method`, `/url`, `/query` o `/protocol` e un modello per filtrare le richieste in base a queste parti specifiche della riga di richiesta della richiesta HTTP. Il filtro degli elementi della riga di richiesta (anziché dell’intera riga di richiesta) è il filtro da preferire.

* **Elementi avanzati della riga di richiesta:** a partire da Dispatcher 4.2.0, sono disponibili quattro nuovi elementi che si possono filtrare. Questi nuovi elementi sono rispettivamente `/path`, `/selectors`, `/extension` e `/suffix`. Includi uno o più di questi elementi per controllare ulteriormente i modelli di URL.

>[!NOTE]
>
>Per ulteriori informazioni sulla parte della riga di richiesta a cui fa riferimento ciascuno di questi elementi, visita la pagina wiki [Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html).

* **Proprietà glob**: la proprietà `/glob` viene utilizzata per la corrispondenza con l’intera riga di richiesta della richiesta HTTP.

>[!CAUTION]
>
>In Dispatcher, il filtro glob è obsoleto. Di conseguenza, evita di utilizzare la proprietà glob nelle sezioni `/filter`, in quanto potrebbe causare problemi di sicurezza. Quindi, invece di:
>
>`/glob "* *.css *"`
>
>devi utilizzare
>
>`/url "*.css"`

#### La parte riga di richiesta delle richieste HTTP {#the-request-line-part-of-http-requests}

HTTP/1.1 definisce la [riga di richiesta](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) come segue:

`Method Request-URI HTTP-Version<CRLF>`

I caratteri `<CRLF>` rappresentano un ritorno a capo seguito da un avanzamento riga. L’esempio che segue è la riga di richiesta ricevuta quando un cliente richiede la pagina in inglese US del sito WKND:

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

I modelli devono tenere conto dei caratteri di spaziatura nella riga di richiesta e dei caratteri `<CRLF>`.

#### Virgolette e apici {#double-quotes-vs-single-quotes}

Quando crei le regole del filtro, utilizza le virgolette `"pattern"` per i modelli semplici. Se utilizzi Dispatcher 4.2.0 o versioni successive e il modello include un’espressione regolare, devi racchiudere il modello regex `'(pattern1|pattern2)'` tra apici.

#### Espressioni regolari {#regular-expressions}

Nelle versioni di Dispatcher successive alla versione 4.2.0, è possibile includere nei modelli dei filtri le espressioni regolari estese POSIX.

#### Risoluzione dei problemi dei filtri {#troubleshooting-filters}

Se i filtri non si attivano come previsto, abilita la [registrazione della traccia](#trace-logging) in Dispatcher che ti permette di vedere quale filtro intercetta la richiesta.

#### Esempio di filtro: rifiuta tutto {#example-filter-deny-all}

Il seguente esempio di sezione di filtro induce Dispatcher a rifiutare tutte le richieste di file. È necessario negare l’accesso a tutti i file e quindi consentire l’accesso ad aree specifiche.

```xml
  /0001  { /glob "*" /type "deny" }
```

Le richieste inviate a un’area negata in modo esplicito causano la restituzione del codice di errore 404 (pagina non trovata).

#### Esempio di filtro: nega l’accesso ad aree specifiche {#example-filter-deny-access-to-specific-areas}

I filtri consentono anche di negare l’accesso a vari elementi, ad esempio pagine ASP e aree sensibili all’interno di un’istanza Publish. Il filtro che segue nega l’accesso alle pagine ASP:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Esempio di filtro: abilita le richieste POST {#example-filter-enable-post-requests}

Il filtro di esempio che segue consente l’inoltro di dati di moduli tramite il metodo POST:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Esempio di filtro: consenti l’accesso alla console del flusso di lavoro {#example-filter-allow-access-to-the-workflow-console}

L’esempio che segue mostra un filtro utilizzato per negare l’accesso esterno alla console del flusso di lavoro:

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

Se la tua istanza Publish utilizza il contesto di un’applicazione web (ad esempio, per la pubblicazione), anche questa condizione può essere aggiunta alla definizione del filtro.

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

Se hai comunque bisogno di accedere a singole pagine all’interno dell’area riservata, puoi consentire l’accesso a quelle pagine. Ad esempio, per consentire l’accesso alla scheda Archivio nella console del flusso di lavoro, aggiungi la seguente sezione:

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>Quando a una richiesta sono applicati più modelli di filtri, l’ultimo modello applicato è quello valido.

#### Esempio di filtro: utilizzo di espressioni regolari {#example-filter-using-regular-expressions}

Questo filtro abilita le estensioni nelle directory di contenuto non pubblico utilizzando un’espressione regolare, qui definita qui tra apici:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Esempio di filtro: filtro per elementi aggiuntivi dell’URL di una richiesta {#example-filter-filter-additional-elements-of-a-request-url}

Di seguito è riportato un esempio di regola che blocca l’acquisizione del contenuto dal percorso `/content` e dalla relativa sottostruttura, utilizzando i filtri per percorso, selettori ed estensioni:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### Esempio di sezione /filter {#example-filter-section}

Quando configuri Dispatcher, devi limitare il più possibile l’accesso esterno. L’esempio che segue fornisce un accesso minimo per i visitatori esterni:

* `/content`
* Contenuti vari, quali progetti e librerie client; ad esempio:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

Dopo aver creato i filtri, [verifica l’accesso alla pagina](#testing-dispatcher-security) per garantire la protezione dell’istanza di AEM.

La seguente sezione `/filter` del file `dispatcher.any` può essere utilizzata come base nel [file di configurazione di Dispatcher.](#dispatcher-configuration-files)

Questo esempio si basa sul file di configurazione predefinito fornito con Dispatcher ed destinato all’utilizzo in un ambiente di produzione. Gli elementi con il prefisso `#` vengono disattivati (esclusi). Fai molta attenzione se decidi di attivarli (rimuovendo il carattere `#` dalla riga corrispondente), in quanto ciò può avere un impatto sulla sicurezza.

Devi prima negare l’accesso a tutto, quindi consentire l’accesso a elementi specifici (limitati):

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
>Se utilizzato con Apache, progetta i modelli di URL del filtro in base alla proprietà DispatcherUseProcessedURL del modulo Dispatcher. (Vedi [Server web Apache - configurare il server web Apache per Dispatcher](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher).)

>[!NOTE]
>
>I filtri `0030` e `0031` relativi a Dynamic Media sono applicabili ad AEM 6.0 e versioni successive.

Se scegli di estendere l’accesso, considera le seguenti raccomandazioni:

* L’accesso esterno a `/admin` deve essere sempre *completamente* disabilitato, se utilizzi CQ versione 5.4 o una versione precedente.

* Fai molta attenzione quando consenti l’accesso ai file in `/libs`. L’accesso deve essere consentito su base individuale.
* Nega l’accesso alla configurazione di replica, in modo che non possa essere visualizzata:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Nega l’accesso al proxy inverso di Google Gadgets:

   * `/libs/opensocial/proxy*`

A seconda dell’installazione, ci potrebbero essere risorse aggiuntive in `/libs`, `/apps` o altrove che devono essere rese disponibili. Puoi utilizzare il file `access.log` come metodo per determinare le risorse a cui si accede esternamente.

>[!CAUTION]
>
>L’accesso alle console e alle directory può rappresentare un rischio per la sicurezza degli ambienti di produzione. A meno che tu non abbia esplicite giustificazioni, devono rimanere disattivati (esclusi).

>[!CAUTION]
>
>Se utilizzi [ report in un ambiente di pubblicazione](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=it#using-reports-in-a-publish-environment), configura Dispatcher per negare l’accesso a `/etc/reports` ai visitatori esterni.

### Limitazione delle stringhe di query {#restricting-query-strings}

A partire dalla versione 4.1.5 di Dispatcher, utilizza la sezione `/filter` per limitare le stringhe di query. Si consiglia vivamente di consentire esplicitamente le stringhe di query ed escludere la tolleranza generica tramite gli elementi del filtro `allow`.

Una singola voce può avere `glob` o una combinazione di `method`, `url`, `query` e `version`, ma non entrambi. L’esempio che segue consente la stringa di query `a=*` e nega tutte le altre stringhe di query per gli URL che si risolvono nel nodo `/etc`:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Se una regola contiene un elemento `/query`, avrà corrispondenza solo con le richieste che contengono una stringa di query e che a loro volta corrispondono al modello di query specificato.
>
>Nell’esempio precedente, se anche le richieste a `/etc` prive di stringa di query devono essere consentite, sono necessarie le seguenti regole:

```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Verifica della sicurezza di Dispatcher {#testing-dispatcher-security}

I filtri di Dispatcher devono bloccare l’accesso alle pagine e agli script seguenti sulle istanze di AEM Publish. Utilizza un browser web per tentare di aprire le pagine seguenti come farebbe un visitatore del sito e verifica che venga restituito il codice 403. Se ottiene qualunque altro risultato, regola i filtri.

Tieni presente che dovresti vedere il normale rendering della pagina per `/content/add_valid_page.html?debug=layout`.

* `/admin`
* `/system/console`
* `/dav/crx.default`
* `/crx`
* `/bin/crxde/logs`
* `/jcr:system/jcr:versionStorage.json`
* `/_jcr_system/_jcr_versionStorage.json`
* `/libs/wcm/core/content/siteadmin.html`
* `/libs/collab/core/content/admin.html`
* `/libs/cq/ui/content/dumplibs.html`
* `/var/linkchecker.html`
* `/etc/linkchecker.html`
* `/home/users/a/admin/profile.json`
* `/home/users/a/admin/profile.xml`
* `/libs/cq/core/content/login.json`
* `/content/../libs/foundation/components/text/text.jsp`
* `/content/.{.}/libs/foundation/components/text/text.jsp`
* `/apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata`
* `/libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet`
* `/content.pages.json`
* `/content.languages.json`
* `/content.blueprint.json`
* `/content.-1.json`
* `/content.10.json`
* `/content.infinity.json`
* `/content.tidy.json`
* `/content.tidy.-1.blubber.json`
* `/content/dam.tidy.-100.json`
* `/content/content/geometrixx.sitemap.txt `
* `/content/add_valid_page.query.json?statement=//*`
* `/content/add_valid_page.qu%65ry.js%6Fn?statement=//*`
* `/content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)`
* `/content/add_valid_path_to_a_page/_jcr_content.json`
* `/content/add_valid_path_to_a_page/jcr:content.json`
* `/content/add_valid_path_to_a_page/_jcr_content.feed`
* `/content/add_valid_path_to_a_page/jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename._jcr_content.feed`
* `/content/add_valid_path_to_a_page/pagename.jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename.docview.xml`
* `/content/add_valid_path_to_a_page/pagename.docview.json`
* `/content/add_valid_path_to_a_page/pagename.sysview.xml`
* `/etc.xml`
* `/content.feed.xml`
* `/content.rss.xml`
* `/content.feed.html`
* `/content/add_valid_page.html?debug=layout`
* `/projects`
* `/tagging`
* `/etc/replication.html`
* `/etc/cloudservices.html`
* `/welcome`

Esegui il comando sotto riportato su un terminale o in un prompt dei comandi per determinare se l’accesso anonimo in scrittura è abilitato. Non dovresti poter scrivere dati sul nodo.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Esegui il comando sotto riportato su un terminale o in un prompt dei comandi per tentare di invalidare la cache di Dispatcher e accertati di ricevere in risposta il codice 404:

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## Abilitazione dell’accesso agli URL personalizzati {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

Configura Dispatcher per abilitare l’accesso agli URL personalizzati configurati per le tue pagine AEM.

Quando l’accesso agli URL personalizzati è abilitato, Dispatcher chiama periodicamente un servizio che viene eseguito sull’istanza di rendering per ottenere un elenco di URL personalizzati. Dispatcher memorizza l’elenco in un file locale. Quando una richiesta per una pagina viene negata a causa di un filtro nella sezione `/filter`, Dispatcher consulta l’elenco degli URL personalizzati. Se l’URL negato è presente nell’elenco, Dispatcher consente l’accesso all’URL personalizzato.

Per abilitare l’accesso agli URL personalizzati, aggiungi una sezione `/vanity_urls` alla sezione `/farms`, simile all’esempio che segue:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

La sezione `/vanity_urls` contiene le seguenti proprietà:

* `/url`: percorso del servizio URL personalizzato in esecuzione sull’istanza di rendering. Il valore di questa proprietà deve essere `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`: percorso del file locale in cui Dispatcher memorizza l’elenco degli URL personalizzati. Accertati che Dispatcher abbia accesso in scrittura a questo file.
* `/delay`: (secondi) il tempo che intercorre tra le chiamate al servizio URL personalizzato.

>[!NOTE]
>
>Se il rendering è un’istanza di AEM, è necessario installare il pacchetto [VanityURLS-Components da Software Distribution](https://experience.adobe.com/#/downloads/content/software-distribution/it/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) per abilitare il servizio URL personalizzato. (Per ulteriori informazioni, vedi [Distribuzione di software](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=it#software-distribution)).

Utilizza la procedura seguente per abilitare l’accesso agli URL personalizzati.

1. Se il servizio di rendering è un’istanza di AEM, installa il pacchetto `com.adobe.granite.dispatcher.vanityurl.content` sull’istanza Publish (vedi la nota precedente).
1. Per ogni URL personalizzato configurato per una pagina AEM o CQ, accertati che la configurazione [`/filter`](#configuring-access-to-content-filter) neghi l’URL. Se necessario, aggiungi un filtro che nega l’URL.
1. Aggiungi la sezione `/vanity_urls` sotto `/farms`.
1. Riavvia il server web Apache.

## Inoltro delle richieste di distribuzione del contenuto (syndication): /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

Le richieste di distribuzione del contenuto sono solitamente destinate solo a Dispatcher, pertanto per impostazione predefinita non vengono inviate al renderer (ad esempio, un’istanza di AEM).

Se necessario, imposta la proprietà `/propagateSyndPost` su `"1"` per inoltrare le richieste di distribuzione del contenuto a Dispatcher. Se impostata, accertati che le richieste POST non siano negate nella sezione del filtro.

## Configurazione della cache di Dispatcher: /cache {#configuring-the-dispatcher-cache-cache}

La sezione `/cache` controlla come Dispatcher memorizza in cache i documenti. Configura diverse sottoproprietà per implementare le strategie di caching:

* `/docroot`
* `/statfile`
* `/serveStaleOnError`
* `/allowAuthorized`
* `/rules`
* `/statfileslevel`
* `/invalidate`
* `/invalidateHandler`
* `/allowedClients`
* `/ignoreUrlParams`
* `/headers`
* `/mode`
* `/gracePeriod`
* `/enableTTL`

Un esempio di sezione cache potrebbe essere il seguente:

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
>Per il caching sensibile alle autorizzazioni, leggere [Caching di contenuto protetto](permissions-cache.md).

### Specifica della directory della cache {#specifying-the-cache-directory}

La proprietà `/docroot` identifica la directory in cui sono archiviati i file memorizzati in cache.

>[!NOTE]
>
>Il valore deve corrispondere esattamente al percorso della directory principale del documento del server web, in modo che Dispatcher e il server web gestiscano gli stessi file.\
>Il server web è responsabile della distribuzione del codice di stato corretto quando viene utilizzato il file di cache di Dispatcher, per questo è importante che possa trovarlo.

Se utilizzi più farm, ogni farm deve utilizzare una radice documento diversa.

### Denominazione dello statfile {#naming-the-statfile}

La proprietà `/statfile` identifica il file da utilizzare come statfile. Dispatcher utilizza questo file per registrare l’ora dell’aggiornamento di contenuto più recente. Lo statfile può essere qualsiasi file sul server web.

Lo statfile non ha contenuto. Quando il contenuto viene aggiornato, Dispatcher aggiorna la marca temporale. Lo statfile predefinito è `.stat` ed è memorizzato nella directory principale dei documenti. Dispatcher blocca l’accesso allo statfile.

>[!NOTE]
>
>Se `/statfileslevel` è configurato, Dispatcher ignora la proprietà `/statfile` e utilizza `.stat` come nome del file.

### Distribuzione di documenti obsoleti in caso di errori {#serving-stale-documents-when-errors-occur}

La proprietà `/serveStaleOnError` definisce se Dispatcher deve restituire i documenti invalidati quando il server di rendering restituisce un errore. Per impostazione predefinita, quando uno statfile viene toccato e invalida il contenuto della cache, Dispatcher elimina il contenuto della cache alla successiva richiesta.

Se `/serveStaleOnError` è impostato su `"1"`, Dispatcher non elimina il contenuto invalidato dalla cache, a meno che il server di rendering non restituisca una risposta corretta. Una risposta 5xx da AEM o un timeout della connessione fa in modo che Dispatcher distribuisca il contenuto obsoleto e risponda con e lo stato HTTP 111 (riconvalida non riuscita).

### Caching nel momento in cui viene utilizzata l’autenticazione {#caching-when-authentication-is-used}

La proprietà `/allowAuthorized` definisce se le richieste che contengono una delle seguenti informazioni di autenticazione devono essere memorizzate in cache:

* L’intestazione `authorization`
* Un cookie denominato `authorization`
* Un cookie denominato `login-token`

Per impostazione predefinita, le richieste che includono queste informazioni di autenticazione non vengono memorizzate in cache, perché l’autenticazione non viene eseguita quando un documento memorizzato in cache viene restituito al client. Questa configurazione impedisce a Dispatcher di distribuire i documenti memorizzati in cache a utenti che non dispongono dei diritti necessari.

Tuttavia, se i tuoi requisiti consentono la memorizzazione in cache dei documenti autenticati, imposta `/allowAuthorized` su uno:

`/allowAuthorized "1"`

>[!NOTE]
>
>Per abilitare la gestione delle sessioni (utilizzando la proprietà `/sessionmanagement`), la proprietà `/allowAuthorized` deve essere impostata su `"0"`.

### Specifica dei documenti da memorizzare in cache {#specifying-the-documents-to-cache}

La proprietà `/rules` definisce quali documenti vengono memorizzati in cache in base al percorso del documento. Indipendentemente dalla proprietà `/rules`, Dispatcher non memorizza mai in cache un documento nelle seguenti circostanze:

* Se l’URI della richiesta contiene un punto interrogativo (`?`).
   * In genere, ciò indica una pagina dinamica, ad esempio il risultato di una ricerca che non richiede di essere memorizzata in cache.
* Se manca l’estensione del file.
   * Il server web ha bisogno dell’estensione per determinare il tipo di documento (tipo MIME).
* Se l’intestazione di autenticazione è impostata(configurabile).
* Se l’istanza AEM risponde con le seguenti intestazioni:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>Dispatcher può memorizzare in cache i metodi GET o HEAD (per l’intestazione HTTP). Per ulteriori informazioni sul caching delle intestazioni di risposta, vedi la sezione [Caching delle intestazioni di risposta HTTP](#caching-http-response-headers).

Ciascun elemento della proprietà `/rules` include un modello [`glob`](#designing-patterns-for-glob-properties) e un tipo:

* Il modello `glob` viene utilizzato per la corrispondenza al percorso del documento.
* Il tipo definisce se memorizzare in cache i documenti che corrispondono al modello `glob`. Il valore può essere consenti (per memorizzare in cache il documento) o nega (per eseguire sempre il rendering del documento).

Se non hai pagine dinamiche (oltre a quelle già escluse dalle regole di cui sopra), puoi configurare Dispatcher per memorizzare tutto in cache. La sezione delle regole si presenta così:

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

Per informazioni sulle proprietà glob, vedi [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

Se alcune sezioni della pagina sono dinamiche (ad esempio, un’applicazione di notizie) o all’interno di un gruppo di utenti chiuso, puoi definire le eccezioni:

>[!NOTE]
>
>I gruppi di utenti chiusi non devono essere memorizzati in cache, perché i loro diritti non vengono verificati per le pagine memorizzate in cache.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**Compressione**

Sui server web Apache è possibile comprimere i documenti memorizzati in cache. La compressione consente ad Apache di restituire il documento in formato compresso, se richiesto dal client. La compressione viene eseguita automaticamente abilitando il modulo Apache `mod_deflate`, ad esempio:

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

### Annullamento della validità dei file per livello di cartella {#invalidating-files-by-folder-level}

Utilizza la proprietà `/statfileslevel` per annullare la validità dei file memorizzati in cache in base al loro percorso:

* Dispatcher crea file `.stat` in ogni cartella, partendo dalla cartella principale dei documenti fino al livello specificato. La cartella principale dei documenti è al livello 0.
* I file vengono invalidati toccando il file `.stat`. L’ultima data di modifica del file `.stat` viene confrontata con l’ultima data di modifica di un documento memorizzato in cache. Il documento viene recuperato, se il file `.stat` è più recente.

* Quando un file che si trova a un determinato livello viene invalidato, vengono toccati **tutti** i file `.stat` a partire dalla directory principale dei documenti **fino al** livello del file invalidato o della proprietà `statsfilevel` configurata (a seconda di quale dei due valori è più basso).

   * Ad esempio: se imposti la proprietà `statfileslevel` su 6 e un file viene invalidato al livello 5, verrà toccato ogni file `.stat` a partire dalla directory principale dei documenti fino al livello 5. Continuando con questo esempio, se un file viene invalidato al livello 7, ogni file .`stat` a partire dalla directory principale dei documenti al livello 6 verrà toccato (in quanto `/statfileslevel = "6"`).

Sono interessate solo le risorse **lungo il percorso** del file invalidato. Prendi in considerazione l’esempio seguente: un sito web utilizza la struttura `/content/myWebsite/xx/.`. Se imposti `statfileslevel` su 3, viene creato un file `.stat` come segue:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

Quando un file in `/content/myWebsite/xx` viene invalidato, viene toccato ogni file `.stat` a partire dalla directory principale dei documenti in giù fino a `/content/myWebsite/xx`. Questo vale solo per `/content/myWebsite/xx` e non per, ad esempio, `/content/myWebsite/yy` oppure `/content/anotherWebSite`.

>[!NOTE]
>
>L’annullamento della validità può essere impedito inviando un’ulteriore intestazione `CQ-Action-Scope:ResourceOnly`. Questo metodo può essere utilizzato per effettuare il flushing di determinate risorse senza invalidare altre parti della cache. Per ulteriori informazioni, vedi [questa pagina](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) e [Annullamento manuale della validità della cache di Dispatcher](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=it#configuring).

>[!NOTE]
>
>Se si specifica un valore per la proprietà `/statfileslevel`, la proprietà `/statfile` viene ignorata.

### Annullamento automatico della validità dei file memorizzati in cache {#automatically-invalidating-cached-files}

La proprietà `/invalidate` definisce i documenti che vengono invalidati automaticamente quando il contenuto viene aggiornato.

Con l’annullamento automatico della validità, Dispatcher non elimina i file memorizzati in cache dopo un aggiornamento del contenuto, ma ne verifica la validità quando vengono successivamente richiesti. I documenti nella cache che non vengono invalidati automaticamente rimarranno nella cache fino a quando un aggiornamento del contenuto non li eliminerà esplicitamente.

L’annullamento automatico della validità viene in genere utilizzato per le pagine HTML. Le pagine HTML spesso contengono collegamenti ad altre pagine, rendendo difficile stabilire se un aggiornamento del contenuto influisce su una determinata pagina. Per fare in modo che tutte le pagine pertinenti vengano invalidate quando il contenuto viene aggiornato, conviene invalidare automaticamente tutte le pagine HTML. La configurazione che segue invalida tutte le pagine HTML:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Per informazioni sulle proprietà glob, vedi [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

Questa configurazione causa la seguente attività quando `/content/wknd/us/en` è attivato:

* Tutti i file con pattern en.*vengono rimossi dalla cartella `/content/wknd/us`.
* La cartella `/content/wknd/us/en./_jcr_content` viene rimossa.
* Tutti gli altri file che corrispondono alla configurazione `/invalidate` non vengono eliminati immediatamente. Questi file vengono eliminati al momento della richiesta successiva. Nel nostro esempio, la pagina `/content/wknd.html` non viene eliminata subito, ma verrà eliminata nel momento in cui `/content/wknd.html` verrà richiesta.

Se offri file PDF e ZIP generati automaticamente per il download, potresti dover annullare automaticamente anche questi. Esempio di configurazione:

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

L’integrazione AEM con Adobe Analytics fornisce i dati di configurazione in un file `analytics.sitecatalyst.js` del sito web. Il file di esempio `dispatcher.any` fornito con Dispatcher include la seguente regola di invalidazione per questo file:

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### Utilizzo di script di invalidazione personalizzati {#using-custom-invalidation-scripts}

La proprietà `/invalidateHandler` ti consente di definire uno script che viene richiamato per ogni richiesta di invalidazione ricevuta da Dispatcher.

Lo script viene richiamato con i seguenti argomenti:

* Handle: il percorso del contenuto invalidato
* Azione: l’azione di replica (ad esempio Attiva, Disattiva)
* Ambito azione: ambito dell’azione di replica (vuoto, a meno che non venga inviata un’intestazione `CQ-Action-Scope: ResourceOnly`, per ulteriori dettagli, vedi [Annullamento della validità delle pagine nella cache da AEM](page-invalidate.md))

Può essere utilizzato per coprire diversi casi d’uso, ad esempio per invalidare cache specifiche per altre applicazioni o per gestire casi in cui l’URL esternalizzato di una pagina e la sua posizione nella directory principale dei documenti non corrispondono al percorso del contenuto.

Di seguito è riportato un esempio di script che registra in un file ogni richiesta di annullamento della validità.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### Esempio di script dell’handler di annullamento della validità {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limitazione dei client che possono effettuare il flushing della cache {#limiting-the-clients-that-can-flush-the-cache}

La proprietà `/allowedClients` definisce client specifici che sono autorizzati a effettuare il flushing della cache. I modelli globbing vengono confrontati con l’IP.

Il seguente esempio:

1. Nega l’accesso a qualsiasi client
1. Consente esplicitamente l’accesso a localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

Per informazioni sulle proprietà glob, vedi [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>È consigliabile definire il valore `/allowedClients`.
>
>In caso contrario, qualsiasi client può inviare una chiamata per cancellare la cache; se ciò avviene ripetutamente, si può avere un forte impatto sulle prestazioni del sito.

### Ignorare i parametri URL {#ignoring-url-parameters}

La sezione `ignoreUrlParams` definisce quali parametri URL vengono ignorati quando si stabilisce se una pagina viene memorizzata in cache o viene distribuita dalla cache:

* Quando un URL di richiesta contiene parametri tutti ignorati, la pagina viene memorizzata in cache.
* Quando un URL di richiesta contiene uno o più parametri che non vengono ignorati, la pagina non viene memorizzata in cache.

Quando un parametro viene ignorato per una pagina, questa pagine viene memorizzata in cache la prima volta che viene richiesta. Alle successive richieste per quella pagina viene distribuita la pagina dalla cache, indipendentemente dal valore del parametro nella richiesta.

Per specificare quali parametri vengono ignorati, aggiungi regole glob alla proprietà `ignoreUrlParams`:

* Per ignorare un parametro, crea una proprietà glob che consenta il parametro.
* Per evitare che la pagina venga memorizzata in cache, crea una proprietà glob che nega il parametro.

L’esempio che segue fa in modo che Dispatcher ignori il parametro `q`, pertanto gli URL di richiesta che includono il parametro q vengono memorizzati in cache:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

Utilizzando il valore di esempio `ignoreUrlParams`, la seguente richiesta HTTP causa la memorizzazione in cache della pagina, perché il parametro `q` viene ignorato:

```xml
GET /mypage.html?q=5
```

Utilizzando il valore di esempio `ignoreUrlParams`, la seguente richiesta HTTP fa in modo che la pagina **non** venga memorizzata in cache, perché il parametro `p` non viene ignorato:

```xml
GET /mypage.html?q=5&p=4
```

Per informazioni sulle proprietà glob, vedi [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

### Caching delle intestazioni di risposta HTTP {#caching-http-response-headers}

>[!NOTE]
>
>Questa funzione è disponibile con la versione **4.1.11** di Dispatcher.

La proprietà `/headers` ti consente di definire i tipi di intestazioni HTTP che verranno memorizzate in cache da Dispatcher. Alla prima richiesta di una risorsa non memorizzata in cache, tutte le intestazioni che corrispondono a uno dei valori configurati (vedi l’esempio di configurazione sotto riportato) vengono memorizzate in un file separato, accanto al file della cache. Alle successive richieste della risorsa memorizzata in cache, le intestazioni memorizzate vengono aggiunte alla risposta.

Di seguito è riportato un esempio di configurazione predefinita:

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
>Considera anche che i caratteri globbing di file non sono consentiti. Per ulteriori dettagli, vedi [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Se hai bisogno che Dispatcher memorizzi e distribuisca le intestazioni di risposta ETag da AEM, procedi come segue:
>
>* Aggiungi il nome dell’intestazione nella sezione `/cache/headers`.
>* Aggiungi la seguente [direttiva Apache](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) nella sezione relativa a Dispatcher:
>
>```xml
>FileETag none
>```

### Autorizzazioni per i file della cache di Dispatcher {#dispatcher-cache-file-permissions}

La proprietà `mode` specifica quali autorizzazioni di file vengono applicate alle nuove directory e ai nuovi file nella cache. Questa impostazione è limitata dalla proprietà `umask` del processo chiamante. Si tratta di un numero ottale costruito dalla somma di uno o più dei seguenti valori:

* `0400` Consenti la lettura per proprietario.
* `0200` Consenti la scrittura per proprietario.
* `0100` Consenti al proprietario di eseguire ricerche nelle directory.
* `0040` Consenti la lettura per membri del gruppo.
* `0020` Consenti la scrittura per membri del gruppo.
* `0010` Consenti ai membri del gruppo di eseguire ricerche nella directory.
* `0004` Consenti la lettura ad altri.
* `0002` Consenti la scrittura ad altri.
* `0001` Consenti ad altri di eseguire ricerche nella directory.

Il valore predefinito è `0755` che consente al proprietario di leggere, scrivere o eseguire ricerche e al gruppo e ad altri di leggere o eseguire ricerche.

### Limitazione del tocco del file .stat {#throttling-stat-file-touching}

Con la proprietà `/invalidate` predefinita, ogni attivazione invalida efficacemente tutti i file `.html` (quando il loro percorso corrisponde alla sezione `/invalidate`). Su un sito web con traffico considerevole, più attivazioni successive incrementeranno il carico della CPU sul backend. In questo caso, sarebbe auspicabile “limitare” il tocco del file `.stat` per mantenere reattivo il sito web. Per farlo, utilizza la proprietà `/gracePeriod`.

La proprietà `/gracePeriod` definisce il numero di secondi in cui una risorsa obsoleta e invalidata automaticamente può ancora essere distribuita dalla cache dopo l’ultima attivazione. La proprietà può essere utilizzata in una configurazione in cui, altrimenti, un batch di attivazioni annullerebbe ripetutamente l’intera cache. Il valore consigliato è 2 secondi.

Per ulteriori informazioni, vedi anche le precedenti sezioni `/invalidate` e `/statfileslevel`.

### Configurazione dell’annullamento della validità della cache basata sul tempo: /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

Se impostata, la proprietà `/enableTTL` valuterà le intestazioni di risposta provenienti dal backend e, se contengono un’età massima `Cache-Control` oppure una data di scadenza `Expires`, viene creato un file ausiliario vuoto accanto al file della cache, con un tempo di modifica uguale alla data di scadenza. Quando il file memorizzato in cache viene richiesto in un tempo superiore a quello di modifica, quel file viene automaticamente richiesto nuovamente dal backend.

>[!NOTE]
>
>Questa funzione è disponibile nella versione **4.1.11** o successive di Dispatcher.

## Configurazione del bilanciamento del carico: /statistics {#configuring-load-balancing-statistics}

La sezione `/statistics` definisce categorie di file per i quali Dispatcher valuta la reattività di ciascun rendering. Dispatcher utilizza i punteggi per determinare a quale rendering inviare una richiesta.

Ogni categoria creata definisce un modello glob. Dispatcher confronta l’URI del contenuto richiesto con questi modelli per determinare la categoria del contenuto richiesto:

* L’ordine delle categorie determina l’ordine in cui vengono confrontate con l’URI.
* Il primo modello di categoria che corrisponde all’URI è la categoria del file. Non vengono valutati altri modelli di categoria.

Dispatcher supporta un massimo di 8 categorie di statistiche. Se definisci più di 8 categorie, vengono utilizzate solo le prime 8.

**Selezione rendering**

Ogni volta che Dispatcher richiede una pagina sottoposta a rendering, utilizza il seguente algoritmo per selezionare il rendering:

1. Se la richiesta contiene il nome del rendering in un cookie `renderid`, Dispatcher lo utilizza.
1. Se la richiesta non include un cookie `renderid`, Dispatcher confronta le statistiche dei rendering:

   1. Dispatcher determina la categoria dell’URI della richiesta.
   1. Dispatcher determina quale rendering ha il punteggio di risposta più basso per quella categoria e lo seleziona.

1. Se non è stato ancora selezionato un rendering, utilizza il primo rendering dell’elenco.

Il punteggio per la categoria di un rendering si basa sui tempi di risposta precedenti, nonché sui precedenti tentativi di connessioni non riusciti e riusciti effettuati da Dispatcher. Per ogni tentativo, viene aggiornato il punteggio per la categoria dell’URI richiesto.

>[!NOTE]
>
>Se non utilizzi il bilanciamento del carico, puoi saltare questa sezione.

### Definizione delle categorie di statistiche {#defining-statistics-categories}

Definire una categoria per ciascun tipo di documento del quale vuoi conservare le statistiche per la selezione del rendering. La sezione `/statistics` contiene una sezione `/categories`. Per definire una categoria, aggiungi una riga sotto la sezione `/categories` con il seguente formato:

`/name { /glob "pattern"}`

Il nome della categoria (`name`) deve essere univoco per la farm. La sezione `pattern` è descritta nella sezione [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

Per determinare la categoria di un URI, Dispatcher lo confronta con ciascun modello di categoria fino a quando non viene trovata una corrispondenza. Dispatcher inizia con la prima categoria dell’elenco e continua seguendo l’ordine. Quindi, inserisci prima le categorie con i modelli più specifici.

Ad esempio, il file predefinito di Dispatcher `dispatcher.any` definisce una categoria HTML e anche un’altra categoria. Tuttavia, la categoria HTML è più specifica e quindi viene visualizzata prima:

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

L’esempio che segue include anche una categoria per le pagine di ricerca:

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

### Indisponibilità del server riflessa nelle statistiche di Dispatcher {#reflecting-server-unavailability-in-dispatcher-statistics}

La proprietà `/unavailablePenalty` imposta il tempo (in decimi di secondo) applicato alle statistiche di rendering quando una connessione al rendering non riesce. Dispatcher aggiunge il tempo alla categoria di statistiche corrispondente all’URI richiesto.

Ad esempio, la penalità viene applicata quando non è possibile stabilire la connessione TCP/IP al nome host/porta designato, perché AEM non è in esecuzione (e non è in ascolto) o a causa di un problema relativo alla rete.

La proprietà `/unavailablePenalty` è un figlio diretto della sezione `/farm` (un elemento di pari livello della sezione `/statistics`).

Se non esiste alcuna proprietà `/unavailablePenalty`, viene utilizzato il valore `"1"`.

```xml
/unavailablePenalty "1"
```

## Identificazione di una cartella di connessione permanenti: /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

La proprietà `/stickyConnectionsFor` definisce una cartella contenente documenti permanenti; la cartella sarà accessibile tramite l’URL. Dispatcher invia tutte le richieste, provenienti da un singolo utente, che sono in questa cartella alla stessa istanza di rendering. Le connessioni permanenti garantiscono la presenza e la coerenza dei dati della sessione per tutti i documenti. Questo meccanismo utilizza il cookie `renderid`.

L’esempio che segue definisce una connessione fissa alla cartella /products:

```xml
/stickyConnectionsFor "/products"
```

Quando una pagina è composta da contenuto proveniente da più nodi di contenuto, includi la proprietà `/paths` che elenca i percorsi del contenuto. Ad esempio, una pagina include il contenuto di `/content/image`, `/content/video` e `/var/files/pdfs`. La seguente configurazione abilita connessioni permanenti per tutto il contenuto della pagina:

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

Quando le connessioni permanenti sono abilitate, il modulo Dispatcher imposta il cookie `renderid`. Questo cookie non ha il flag `httponly`, che però deve essere aggiunto per rafforzare la sicurezza. Per farlo, imposta la proprietà `httpOnly` nel nodo `/stickyConnections` di un file di configurazione `dispatcher.any`. Il valore della proprietà (`0` oppure `1`) definisce se al cookie `renderid` viene aggiunto l’attributo `HttpOnly`. Il valore predefinito è `0`, il che significa che l’attributo non viene aggiunto.

Per ulteriori informazioni sul flag `httponly`, leggi [questa pagina](https://www.owasp.org/index.php/HttpOnly).

### Flag secure {#secure}

Quando le connessioni permanenti sono abilitate, il modulo Dispatcher imposta il cookie `renderid`. Questo cookie non ha il flag `secure`, che però deve essere aggiunto per rafforzare la sicurezza. Per farlo, imposta la proprietà `secure` nel nodo `/stickyConnections` di un file di configurazione `dispatcher.any`. Il valore della proprietà (`0` oppure `1`) definisce se al cookie `renderid` viene aggiunto l’attributo `secure`. Il valore predefinito è `0`, il che significa che l’attributo viene aggiunto **se** la richiesta in ingresso è sicura. Se il valore è impostato su `1`, il flag secure viene aggiunto indipendentemente dal fatto che la richiesta in ingresso sia sicura o meno.

## Gestione degli errori di connessione del rendering {#handling-render-connection-errors}

Configura il comportamento di Dispatcher quando il server di rendering restituisce l’errore 500 o non è disponibile.

### Specifica di una pagina di verifica dello stato di integrità {#specifying-a-health-check-page}

Utilizzare la proprietà `/health_check` per specificare un URL che viene verificato quando viene restituito il codice di stato 500. Se anche questa pagina restituisce il codice di stato 500, l’istanza viene considerata non disponibile e al rendering viene applicata una penalità di tempo configurabile (`/unavailablePenalty`) prima di un nuovo tentativo.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifica del ritardo prima di un nuovo tentativo di richiedere una pagina {#specifying-the-page-retry-delay}

La proprietà `/retryDelay` imposta il tempo (in secondi) che Dispatcher deve attendere prima di ogni successivo turno di tentativi di connessione con i rendering nella farm. Per ogni turno, il numero massimo di tentativi di Dispatcher di connettersi a un rendering è pari al numero di rendering nella farm.

Dispatcher utilizza il valore `"1"`, se `/retryDelay` non è definito in modo esplicito. Il valore predefinito è appropriato nella maggior parte dei casi.

```xml
/retryDelay "1"
```

### Configurazione del numero di tentativi {#configuring-the-number-of-retries}

La proprietà `/numberOfRetries` imposta il numero massimo di turni di tentativi di connessione ai rendering effettuati da Dispatcher. Se Dispatcher non riesce a connettersi a un rendering dopo questo numero di tentativi, restituisce una risposta di esito negativo.

Per ogni turno, il numero massimo di tentativi di Dispatcher di connettersi a un rendering è pari al numero di rendering nella farm. Pertanto, il numero massimo di tentativi di connessione da parte di Dispatcher è (`/numberOfRetries`) x (il numero di rendering).

Se il valore non è definito in modo esplicito, il valore predefinito è `5`.

```xml
/numberOfRetries "5"
```

### Utilizzo del meccanismo di failover {#using-the-failover-mechanism}

Abilita il meccanismo di failover nella farm di Dispatcher per inviare nuovamente le richieste a rendering diversi quando la richiesta originale non riesce. Quando il failover è abilitato, Dispatcher si comporta come segue:

* Quando una richiesta inviata a un rendering restituisce lo stato HTTP 503 (NON DISPONIBILE), Dispatcher invia la richiesta a un altro rendering.
* Quando una richiesta inviata a un rendering restituisce lo stato HTTP 50x (diverso da 503), Dispatcher invia una richiesta per la pagina configurata per la proprietà `health_check`.
   * Se la verifica dello stato di integrità restituisce il codice 500 (INTERNAL_SERVER_ERROR), Dispatcher invia la richiesta originale a un altro rendering.
   * Se la verifica dello stato di integrità restituisce il codice HTTP 200, Dispatcher restituisce il codice di errore HTTP 500 iniziale al client.

Per abilitare il failover, aggiungi la seguente riga alla farm (o al sito web):

```xml
/failover "1"
```

>[!NOTE]
>
>Per ritentare le richieste HTTP che contengono un corpo, Dispatcher invia un’intestazione di richiesta `Expect: 100-continue` al rendering prima di eseguire lo spooling del contenuto effettivo. CQ 5.5 con CQSE risponde immediatamente con il codice di stato 100 (CONTINUA) o con un codice di errore. Anche altri contenitori servlet dovrebbero supportare questa funzione.

## Ignorare gli errori di interruzione: /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Questa opzione di solito non è necessaria. Devi utilizzarla solo quando vedi i seguenti messaggi nel registro:
>
>`Error while reading response: Interrupted system call`

Qualsiasi chiamata di sistema orientata al file system può essere interrotta `EINTR` se l’oggetto della chiamata di sistema si trova su un sistema remoto accessibile tramite NFS. Se queste chiamate di sistema possono andare in timeout o essere interrotte dipende da come il file system sottostante è stato installato sul computer locale.

Usa il parametro `/ignoreEINTR`, se la tua istanza ha questa configurazione e il registro contiene il seguente messaggio:

`Error while reading response: Interrupted system call`

Internamente, Dispatcher legge la risposta dal server remoto (cioè, da AEM) utilizzando un loop che può essere rappresentato come:

```text
while (response not finished) {  
read more data  
}
```

Questi messaggi possono essere generati quando la proprietà `EINTR` è inserita nella sezione “`read more data`” e sono dovuti alla ricezione di un segnale prima della ricezione di dati.

Per ignorare queste interruzioni, puoi aggiungere il seguente parametro a `dispatcher.any` (prima di `/farms`):

`/ignoreEINTR "1"`

Se si imposta `/ignoreEINTR` su `"1"`, Dispatcher continua a tentare di leggere i dati fino a quando non viene letta la risposta completa. Il valore predefinito è `0` e disattiva l’opzione.

## Progettazione di modelli per le proprietà glob {#designing-patterns-for-glob-properties}

Diverse sezioni nel file di configurazione di Dispatcher utilizzano le proprietà `glob` come criteri di selezione per le richieste client. I valori delle proprietà `glob` sono modelli che Dispatcher confronta con un aspetto della richiesta, ad esempio il percorso della risorsa richiesta o l’indirizzo IP del client. Ad esempio, gli elementi nella sezione `/filter` utilizzano i modelli `glob` per identificare i percorsi delle pagine su cui Dispatcher agisce o che rifiuta.

I valori `glob` possono includere caratteri jolly e caratteri alfanumerici per definire il modello.

| Carattere jolly | Descrizione | Esempi |
|--- |--- |--- |
| `*` | Corrisponde a zero o più istanze contigue di qualsiasi carattere nella stringa. Il carattere finale della corrispondenza è determinato da una delle seguenti situazioni: <br/>Un carattere nella stringa corrisponde al carattere successivo nel modello e il carattere del modello ha le seguenti caratteristiche:<br/><ul><li>Non è un *</li><li>Non è un ?</li><li>Un carattere letterale (incluso uno spazio) o una classe di caratteri.</li><li>Viene raggiunta la fine del modello.</li></ul>All’interno di una classe di caratteri, il carattere viene interpretato letteralmente. | `*/geo*` Corrisponde a qualsiasi pagina sotto i nodi `/content/geometrixx` e `/content/geometrixx-outdoors`. Le seguenti richieste HTTP corrispondono al modello glob: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>Corrisponde a qualsiasi pagina sotto il nodo `/content/geometrixx-outdoors`. Ad esempio, la seguente richiesta HTTP corrisponde al modello glob: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Corrisponde a qualsiasi carattere singolo. Da utilizzare al di fuori delle classi di caratteri. All’interno di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*outdoors/??/*`<br/> Corrisponde alle pagine in qualsiasi lingua del sito geometrixx-outdoors. Ad esempio, la seguente richiesta HTTP corrisponde al modello glob: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>La seguente richiesta non corrisponde al modello glob: <br/><ul><li>“GET /content/geometrixx-outdoors/en.html”</li></ul> |
| `[ and ]` | Richiama l’inizio e la fine di una classe di caratteri. Le classi di caratteri possono includere uno o più intervalli di caratteri e caratteri singoli.<br/>Una corrispondenza si verifica, se il carattere di destinazione corrisponde a uno qualsiasi dei caratteri della classe di caratteri o dei caratteri all’interno di un intervallo definito.<br/>Se si omette la parentesi quadra di chiusura, il modello non produce alcuna corrispondenza. | `*[o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>Corrisponde alle seguenti richieste HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Indica un intervallo di caratteri. Da utilizzare nelle classi di caratteri.  Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[m-p]men.html*` Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Ignora il carattere o la classe di che segue. Da utilizzare solo per ignorare caratteri e intervalli di caratteri all’interno delle classi di caratteri. Equivalente a `^ wildcard`. <br/>Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[!o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` oppure `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Ignora il carattere o l’intervallo di caratteri che segue. Da utilizzare solo per ignorare caratteri e intervalli di caratteri all’interno delle classi di caratteri. Equivalente al carattere jolly `!`. <br/>Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | Gli esempi relativi al carattere jolly `!` valgono, ma previa sostituzione, negli esempi di modelli, dei caratteri `!` con i caratteri `^`. |


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

Nella configurazione del server web, puoi impostare:

* Posizione del file di registro di Dispatcher.
* Livello di registro.

Per ulteriori informazioni, vedi la documentazione del server web e il file readme dell’istanza di Dispatcher.

**Rotazione/piping dei registri di Apache**

Se utilizzi un server web **Apache** puoi utilizzare la funzionalità standard per la rotazione e/o il piping dei registri. Ad esempio, effettuando il piping dei registri:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Si otterrà automaticamente una rotazione:

* Del file di registro di Dispatcher; con una marca temporale nell’estensione (`logs/dispatcher.log%Y%m%d`).
* Su base settimanale (60 x 60 x 24 x 7 = 604800 secondi).

Vedi la documentazione del server web Apache sulla rotazione/piping dei registri; ad esempio, [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Al momento dell’installazione, il livello di registro predefinito è alto (ovvero livello 3 = Debug), in modo che Dispatcher registri tutti gli errori e gli avvisi. Ciò è molto utile nelle fasi iniziali.
>
>Tuttavia, questa impostazione richiede risorse aggiuntive, quindi quando Dispatcher funziona senza problemi *in base alle tue esigenze*, puoi (dovresti) abbassare il livello di registro.

### Registrazione della traccia {#trace-logging}

Tra gli altri miglioramenti apportati a Dispatcher, la versione 4.2.0 introduce anche la funzione di registrazione della traccia.

Si tratta di un livello più alto della registrazione di debug, in quanto mostra informazioni aggiuntive nei registri. La registrazione ora include:

* I valori delle intestazioni inoltrate.
* La regola applicata per una determinata azione.

Puoi abilitare la funzione di registrazione della traccia impostando il livello di registro su `4` nel server web.

Di seguito è riportato un esempio di registri con tracciamento abilitato:

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

## Verifica del funzionamento di base {#confirming-basic-operation}

Per verificare il funzionamento e l’interazione di base del server web, di Dispatcher e dell’istanza di AEM, procedi come segue:

1. Imposta `loglevel` su `3`.

1. Avvia il server web; ciò determina anche l’avvio di Dispatcher.
1. Avvia l’istanza di AEM.
1. Verifica i file di registro e di errore per il server web e Dispatcher.
   * A seconda del server web, dovresti visualizzare messaggi di questo tipo:
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` e
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Naviga nel sito web tramite il server web. Verifica che il contenuto sia visualizzato nel modo corretto.\
   Ad esempio, in un’installazione locale in cui AEM viene eseguito sulla porta `4502` e il server Web sulla porta `80`, accedi alla console dei siti web utilizzando entrambi:
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * Il risultato dovrebbe essere identico. Verifica l’accesso ad altre pagine con lo stesso meccanismo.

1. Verifica che la directory della cache sia stata riempita.
1. Attiva una pagina per verificare che il flushing della cache avvenga correttamente.
1. Se tutto funziona correttamente, puoi ridurre `loglevel` a `0`.

## Utilizzo di più istanze di Dispatcher {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più istanze di Dispatcher. Ad esempio, puoi utilizzare:

* Un’istanza di Dispatcher per pubblicare un sito web nella Intranet.
* Una seconda istanza di Dispatcher, con indirizzo e impostazioni di sicurezza diversi, per pubblicare lo stesso contenuto in Internet.

In questo caso, accertati che ogni richiesta venga gestita tramite un’unica istanza di Dispatcher. Un’istanza di Dispatcher non gestisce le richieste provenienti da un’altra istanza di Dispatcher. Accertati pertanto che entrambe le istanze di Dispatcher accedano direttamente al sito web di AEM.

## Debugging {#debugging}

Quando si aggiunge l’intestazione `X-Dispatcher-Info` a una richiesta, Dispatcher risponde indicando se la destinazione è stata memorizzata in cache, restituita dalla cache o non è memorizzabile in cache. L’intestazione di risposta `X-Cache-Info` contiene queste informazioni in un formato leggibile. Puoi utilizzare queste intestazioni di risposta per eseguire il debug dei problemi relativi alle risposte memorizzate in cache da Dispatcher.

Questa funzionalità non è abilitata per impostazione predefinita, pertanto affinché l’intestazione di risposta `X-Cache-Info` sia inclusa, la farm deve contenere la seguente voce:

```xml
/info "1"
```

Ad esempio:

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

Inoltre, l’intestazione `X-Dispatcher-Info` non ha bisogno di un valore, ma se utilizzi `curl` per il test devi fornire un valore per poter inviare l’intestazione, ad esempio:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

Di seguito è riportato un elenco contenente le intestazioni di risposta che `X-Dispatcher-Info` restituirà:

* **memorizzato in cache**\
   Il file di destinazione è contenuto nella cache e Dispatcher ha stabilito che è valido per la distribuzione.
* **caching**\
   Il file di destinazione non è contenuto nella cache e Dispatcher ha stabilito che è valido per il caching dell’output e per la distribuzione.
* **caching: il file stat è più recente**
Il file di destinazione è contenuto nella cache, tuttavia, viene invalidato da un file stat più recente. Dispatcher eliminerà il file di destinazione, lo ricreerà dall’output e lo distribuirà.
* **non memorizzabile in cache: nessuna directory principale dei documenti**
La configurazione della farm non contiene una directory principale dei documenti (elemento di configurazione 
`cache.docroot`).
* **non memorizzabile in cache: percorso del file della cache troppo lungo**\
   Il file di destinazione, ovvero la concatenazione di directory principale del documento e file URL, supera la lunghezza massima consentita per i nomi di file sul sistema.
* **non memorizzabile in cache: percorso del file temporaneo troppo lungo**\
   Il modello di nome file temporaneo supera la lunghezza massima consentita per i nomi di file sul sistema. Dispatcher crea un file temporaneo prima di creare o sovrascrivere effettivamente il file memorizzato in cache. Il nome del file temporaneo è il nome del file di destinazione con i caratteri `_YYYYXXXXXX` aggiunti alla fine, in cui i caratteri `Y` e `X` verranno sostituiti per creare un nome univoco.
* **non memorizzabile in cache: l’URL della richiesta non ha estensione**\
   L’URL della richiesta non ha un’estensione oppure un percorso segue l’estensione del file, ad esempio: `/test.html/a/path`.
* **non memorizzabile in cache: la richiesta non era GET o HEAD**
Il metodo HTTP non è né GET né HEAD. Dispatcher presume che l’output conterrà dati dinamici che non devono essere memorizzati in cache.
* **non memorizzabile in cache: la richiesta conteneva una stringa di query**\
   La richiesta conteneva una stringa di query. Dispatcher presume che l’output dipende dalla stringa di query specificata e pertanto non la memorizza in cache.
* **non memorizzabile in cache: il gestore di sessione non ha autenticato**\
   La cache della farm è gestita da un gestore di sessione (la configurazione contiene un nodo `sessionmanagement`) e la richiesta non conteneva le informazioni di autenticazione appropriate.
* **non memorizzabile in cache: la richiesta contiene un’autorizzazione**\
   La farm non può memorizzare in cache l’output (`allowAuthorized 0`) e la richiesta contiene informazioni di autenticazione.
* **non memorizzabile in cache: la destinazione è una directory**\
   Il file di destinazione è una directory. Ciò potrebbe indicare un errore concettuale, in cui un URL e un URL secondario contengono entrambi un output memorizzabile in cache, ad esempio se una richiesta a `/test.html/a/file.ext` arriva prima e contiene un output memorizzabile in cache, Dispatcher non potrà memorizzare l’output di una richiesta successiva in `/test.html`.
* **non memorizzabile in cache: l’URL della richiesta ha una barra finale**\
   L’URL della richiesta ha una barra finale.
* **non memorizzabile in cache: URL della richiesta non incluso nelle regole di cache**\
   Le regole di cache della farm negano esplicitamente il caching dell’output di alcuni URL di richiesta.
* **non memorizzabile in cache: accesso negato dal modulo AuthChecker**\
   Il modulo AuthChecker della farm ha negato l’accesso al file memorizzato in cache.
* **non memorizzabile in cache: sessione non valida**
La cache della farm è gestita da un gestore di sessione (la configurazione contiene un nodo `sessionmanagement`) e la sessione dell’utente non è o non è più valida.
* **non memorizzabile in cache: la risposta contiene`no_cache`**
il server remoto ha restituito un’intestazione 
`Dispatcher: no_cache` che vieta a Dispatcher di memorizzare in cache l’output.
* **non memorizzabile in cache: la lunghezza del contenuto della risposta è zero**
La lunghezza del contenuto della risposta è zero; Dispatcher non crea un file di lunghezza zero.
