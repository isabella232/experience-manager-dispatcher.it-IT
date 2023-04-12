---
title: Configurazione di Dispatcher
description: Scopri come configurare Dispatcher. Scopri il supporto per IPv4 e IPv6, i file di configurazione, le variabili di ambiente, la denominazione dell’istanza, la definizione delle farm, l’identificazione degli host virtuali e altro ancora.
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: 434a17077cea8958a55a637eddd1f4851fc7f2ee
workflow-type: tm+mt
source-wordcount: '8941'
ht-degree: 71%

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

* Se il file di configurazione è di grandi dimensioni, puoi suddividerlo in più file più piccoli (che sono più facili da gestire) e includerli ciascuno.
* Per includere i file generati automaticamente.

Ad esempio, per includere il file myFarm.any nella configurazione /farms, utilizza il seguente codice:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Per specificare un intervallo di file da includere, utilizzare l&#39;asterisco (`*`) come carattere jolly.

Ad esempio, se i file da `farm_1.any` a `farm_5.any` contengono la configurazione delle farm da uno a cinque, puoi includerli come segue:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Utilizzo delle variabili di ambiente {#using-environment-variables}

Puoi utilizzare le variabili di ambiente nelle proprietà con valori stringa nel file dispatcher.any, invece di utilizzare valori a codifica fissa (hard-coding). Per includere il valore di una variabile di ambiente, utilizza il formato `${variable_name}`.

Ad esempio, se il file dispatcher.any si trova nella stessa directory della directory cache, il seguente valore per [docroot](#specifying-the-cache-directory) può essere utilizzata:

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
* Diversi altri comportamenti, ad esempio quali file memorizzare in cache e dove memorizzare in cache.

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
>Se utilizzi più di una farm di rendering, l’elenco viene valutato dal basso verso l’alto. Questo flusso è rilevante quando si definisce [Host virtuali](#identifying-virtual-hosts-virtualhosts) per i siti web.

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
>Il parametro `/homepage` (solo IIS) non funziona più. In Alternativa, utilizza [URL Rewrite Module per Microsoft IIS](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Se utilizzi Apache, utilizza il modulo `mod_rewrite`. Per informazioni su `mod_rewrite`, vedi la documentazione del sito web di Apache (ad esempio, [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). Quando si utilizza `mod_rewrite`, è consigliabile utilizzare il flag ‘passthrough|PT’ (passare all’handler successivo) per forzare il motore di riscrittura a impostare il campo `uri` della struttura interna `request_rec` sul valore del campo `filename`.

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
* Rimuovere le intestazioni, ad esempio le intestazioni di autenticazione rilevanti solo per il server web.

Se si personalizza il set di intestazioni da trasmettere, è necessario specificare un elenco completo di intestazioni, comprese quelle che sono normalmente incluse per impostazione predefinita.

Ad esempio, un’istanza di Dispatcher che gestisce le richieste di attivazione delle pagine per le istanze Publish richiede l’intestazione `PATH` nella sezione `/clientheaders`. La `PATH` l’intestazione abilita la comunicazione tra l’agente di replica e il Dispatcher.

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

Nell’esempio seguente la configurazione gestisce le richieste per `.com` e `.ch` domini della mia azienda e tutti i domini della miaSubDivision:

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
* Se no `virtualhosts` i valori `scheme` e `uri` parti che corrispondono `scheme` e `uri` della richiesta, il primo host virtuale rilevato che corrisponde al `host` della richiesta.
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
>`/allowAuthorized` Imposta su `"0"` in `/cache` per abilitare questa funzione. Come descritto nella sezione [Caching quando si utilizza l’autenticazione](#caching-when-authentication-is-used), quando si imposta `/allowAuthorized 0 ` le richieste che includono informazioni di autenticazione **non** vengono memorizzate nella cache. Se viene richiesta la memorizzazione in cache sensibile alle autorizzazioni, consulta la pagina [Caching di contenuto protetto](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/permissions-cache.html?lang=it).

Crea una sessione protetta per l’accesso alla farm di rendering in modo che gli utenti possano accedere a qualsiasi pagina della farm. Una volta effettuato il login, gli utenti possono accedere alle pagine della farm. Per informazioni sull’utilizzo di questa funzione con i gruppi utenti chiusi (CUG), vedi [Creazione di un gruppo utenti chiuso](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=it#creating-the-user-group-to-be-used). Inoltre, vedi [Elenco di controllo della sicurezza](/help/using/security-checklist.md) di Dispatcher prima di andare “live”.

La proprietà `/sessionmanagement` è una sottoproprietà di `/farms`.

>[!CAUTION]
>
>Se le sezioni del sito web utilizzano requisiti di accesso diversi, è necessario definire più farm.

**/sessionmanagement** ha diversi sottoparametri:

**/directory** (obbligatorio)

La directory in cui sono memorizzate le informazioni della sessione. Se la directory non esiste, viene creata.

>[!CAUTION]
>
> Quando si configura il sottoparametro della directory, **non** puntare alla cartella principale (`/directory "/"`) in quanto può causare gravi problemi. Specifica sempre il percorso della cartella in cui sono memorizzate le informazioni sulla sessione. Esempio:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (facoltativo)

Codifica delle informazioni della sessione. Utilizza `md5` per la crittografia, utilizzando l’algoritmo md5 oppure `hex` per la codifica esadecimale. Se crittografi i dati della sessione, un utente con accesso al file system non può leggere il contenuto della sessione. Il valore predefinito è `md5`.

**/header** (facoltativo)

Nome dell’intestazione HTTP o del cookie che memorizza le informazioni di autorizzazione. Se memorizzi le informazioni nell’intestazione http, utilizza `HTTP:<header-name>`. Per memorizzare le informazioni in un cookie, utilizza `Cookie:<header-name>`. Se non si specifica un valore, `HTTP:authorization` viene utilizzato.

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

La seguente sezione /renders di esempio identifica un&#39;istanza AEM eseguita sullo stesso computer di Dispatcher:

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

Specifica il tempo in millisecondi che una risposta può richiedere. Il valore predefinito è `"600000"` e Dispatcher deve attendere 10 minuti. Impostazione di `"0"` elimina il timeout .

Se viene raggiunto il timeout durante l’analisi delle intestazioni di risposta, viene restituito lo stato HTTP 504 (Gateway non valido). Se viene raggiunto il timeout durante la lettura del corpo della risposta, Dispatcher restituisce la risposta incompleta al client. Elimina anche eventuali file di cache che potrebbero essere stati scritti.

**/ipv4**

Specifica se Dispatcher utilizza la funzione `getaddrinfo` (per IPv6) o la funzione `gethostbyname` (per IPv4) per ottenere l’indirizzo IP del rendering. Se si imposta il valore 0, viene utilizzato `getaddrinfo`. Se si imposta il valore `1`, viene utilizzato `gethostbyname`. Il valore predefinito è `0`.

La funzione `getaddrinfo` restituisce un elenco di indirizzi IP. Dispatcher scorre l’elenco degli indirizzi fino a quando non stabilisce una connessione TCP/IP. Pertanto, la proprietà `ipv4` è importante quando il nome host di rendering è associato a più indirizzi IP e l’host, in risposta alla funzione `getaddrinfo`, restituisce un elenco di indirizzi IP che sono sempre nello stesso ordine. In questa situazione, utilizza la funzione `gethostbyname` in modo che l’indirizzo IP con cui Dispatcher si connette sia randomizzato.

L’ELB (Elastic Load Balancing) di Amazon è un servizio che risponde a getaddrinfo con un elenco potenzialmente identico di indirizzi IP.

**/secure**

Se la `/secure` ha un valore di `"1"`, Dispatcher utilizza HTTPS per comunicare con l’istanza AEM. Per ulteriori dettagli, consulta anche [Configurazione di Dispatcher per utilizzare SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Con la versione di Dispatcher **4.1.6**, puoi configurare la proprietà `/always-resolve` come segue:

* Quando è impostato su `"1"`, risolve il nome host su ogni richiesta (Dispatcher non memorizza mai in cache alcun indirizzo IP). Potrebbe verificarsi un leggero impatto sulle prestazioni a causa della chiamata aggiuntiva necessaria per ottenere le informazioni host per ciascuna richiesta.
* Se la proprietà non è impostata, l’indirizzo IP viene memorizzato nella cache per impostazione predefinita.

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

La `/filter` è costituita da una serie di regole che negano o consentono l’accesso al contenuto in base ai pattern presenti nella riga della richiesta HTTP. Utilizza una strategia di inserire nell&#39;elenco Consentiti per il tuo `/filter` sezione:

* Per prima cosa, nega l’accesso a tutto.
* Quindi, consenti l’accesso al contenuto in base alle esigenze.

>[!NOTE]
>
>Elimina la cache ogni volta che si verificano modifiche nelle regole del filtro.

### Definizione di un filtro {#defining-a-filter}

Ogni elemento della sezione `/filter` include un tipo e un modello corrispondenti a un elemento specifico della riga di richiesta o all’intera riga di richiesta. Ciascun filtro può contenere i seguenti elementi:

* **Tipo**: `/type` indica se consentire o negare l’accesso delle richieste che corrispondono al modello. Il valore può essere `allow` oppure `deny`.

* **Elemento della riga di richiesta:** Include `/method`, `/url`, `/query` o `/protocol` e un modello per filtrare le richieste in base a queste parti specifiche della riga di richiesta della richiesta HTTP. Il filtro degli elementi della riga di richiesta (anziché dell’intera riga di richiesta) è il filtro da preferire.

* **Elementi avanzati della riga di richiesta:** a partire da Dispatcher 4.2.0, sono disponibili quattro nuovi elementi che si possono filtrare. Questi nuovi elementi `/path`, `/selectors`, `/extension`e `/suffix` rispettivamente. Includi uno o più di questi elementi per controllare ulteriormente i modelli di URL.

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
>use
>
>`/url "*.css"`

#### La parte riga di richiesta delle richieste HTTP {#the-request-line-part-of-http-requests}

HTTP/1.1 definisce la [riga di richiesta](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) come segue:

`Method Request-URI HTTP-Version<CRLF>`

I caratteri `<CRLF>` rappresentano un ritorno a capo seguito da un avanzamento riga. L’esempio che segue è la riga di richiesta ricevuta quando un cliente richiede la pagina in inglese US del sito WKND:

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

I pattern devono tenere conto degli spazi nella riga della richiesta e nella `<CRLF>` caratteri.

#### Virgolette e apici {#double-quotes-vs-single-quotes}

Quando crei le regole del filtro, utilizza le virgolette `"pattern"` per i modelli semplici. Se utilizzi Dispatcher 4.2.0 o versioni successive e il modello include un’espressione regolare, devi racchiudere il modello regex `'(pattern1|pattern2)'` tra apici.

#### Espressioni regolari {#regular-expressions}

Nelle versioni di Dispatcher successive alla versione 4.2.0, è possibile includere nei modelli dei filtri le espressioni regolari estese POSIX.

#### Risoluzione dei problemi dei filtri {#troubleshooting-filters}

Se i filtri non si attivano come previsto, abilita [Registrazione traccia](#trace-logging) in Dispatcher per vedere quale filtro intercetta la richiesta.

#### Esempio di filtro: rifiuta tutto {#example-filter-deny-all}

Il seguente esempio di sezione di filtro induce Dispatcher a rifiutare tutte le richieste di file. Nega l’accesso a tutti i file e quindi consenti l’accesso a aree specifiche.

```xml
/0001  { /type "deny" /url "*"  }
```

Le richieste inviate a un’area negata in modo esplicito causano la restituzione del codice di errore 404 (pagina non trovata).

#### Esempio di filtro: nega l’accesso ad aree specifiche {#example-filter-deny-access-to-specific-areas}

I filtri consentono inoltre di negare l’accesso a vari elementi, ad esempio pagine ASP e aree sensibili all’interno di un’istanza di pubblicazione. Il filtro che segue nega l’accesso alle pagine ASP:

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

Se l’istanza di pubblicazione utilizza un contesto dell’applicazione web (ad esempio pubblica), può essere aggiunta anche alla definizione del filtro.

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

Se è necessario accedere a singole pagine all’interno dell’area riservata, è possibile accedervi. Ad esempio, per consentire l’accesso alla scheda Archivio nella console del flusso di lavoro, aggiungi la seguente sezione:

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>Quando a una richiesta sono applicati più pattern di filtri, l’ultimo pattern di filtro applicato è valido.

#### Esempio di filtro: utilizzo di espressioni regolari {#example-filter-using-regular-expressions}

Questo filtro abilita le estensioni nelle directory di contenuto non pubblico utilizzando un’espressione regolare, qui definita qui tra apici:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Esempio di filtro: filtro per elementi aggiuntivi dell’URL di una richiesta {#example-filter-filter-additional-elements-of-a-request-url}

Di seguito è riportato un esempio di regola che blocca l’acquisizione del contenuto dal `/content` percorso e relativo sottoalbero, utilizzando i filtri per percorso, selettori ed estensioni:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|jcr:content|_jcr_content|search|childrenlist|ext|assets|assetsearch|[0-9-]+)'
        /extension '(json|xml|html|feed))'
        }
```

### Esempio di sezione /filter {#example-filter-section}

Durante la configurazione di Dispatcher, devi limitare il più possibile l’accesso esterno. L’esempio che segue fornisce un accesso minimo per i visitatori esterni:

* `/content`
* contenuti vari, ad esempio progetti e librerie client. Ad esempio:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

Dopo aver creato i filtri, [accesso alla pagina di prova](#testing-dispatcher-security) per garantire la protezione dell’istanza AEM.

La seguente sezione `/filter` del file `dispatcher.any` può essere utilizzata come base nel [file di configurazione di Dispatcher.](#dispatcher-configuration-files)

Questo esempio si basa sul file di configurazione predefinito fornito con Dispatcher ed destinato all’utilizzo in un ambiente di produzione. Elementi con prefisso `#` sono disattivati (commentato). Presta attenzione se decidi di attivare uno qualsiasi di questi elementi (rimuovendo la `#` su quella linea). Questo può avere un impatto sulla sicurezza.

Nega l&#39;accesso a tutto, quindi consenti l&#39;accesso a elementi specifici (limitati):

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
      /0001  { /type "deny" /url "*"  }

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

<!----
>[!NOTE]
>
>Filters `0030` and `0031` regarding Dynamic Media are applicable to AEM 6.0 and higher. -->

Se scegli di estendere l’accesso, considera le seguenti raccomandazioni:

* Disattiva l&#39;accesso esterno a `/admin` se utilizzi CQ versione 5.4 o precedente.

* Fai molta attenzione quando consenti l’accesso ai file in `/libs`. L’accesso deve essere consentito su base individuale.
* Nega l’accesso alla configurazione di replica, in modo che non possa essere visualizzata:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Nega l’accesso al proxy inverso di Google Gadgets:

   * `/libs/opensocial/proxy*`

A seconda dell’installazione, ci potrebbero essere risorse aggiuntive in `/libs`, `/apps` o altrove che devono essere rese disponibili. Puoi utilizzare il file `access.log` come metodo per determinare le risorse a cui si accede esternamente.

>[!CAUTION]
>
>L’accesso alle console e alle directory può rappresentare un rischio per la sicurezza degli ambienti di produzione. A meno che non si disponga di giustificazioni esplicite, queste devono rimanere disattivate (commentato).

>[!CAUTION]
>
>Se sei [utilizzo dei rapporti in un ambiente di pubblicazione](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=it#using-reports-in-a-publish-environment), è necessario configurare Dispatcher per negare l’accesso a `/etc/reports` per visitatori esterni.

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
>Se una regola contiene `/query`, corrisponde solo alle richieste che contengono una stringa di query e corrispondono al pattern di query fornito.
>
>Nell’esempio precedente, se anche le richieste a `/etc` prive di stringa di query devono essere consentite, sono necessarie le seguenti regole:

```xml
/filter {  
>/0001 { /type "deny" /method "*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type "deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Verifica della sicurezza di Dispatcher {#testing-dispatcher-security}

I filtri di Dispatcher devono bloccare l’accesso alle pagine e agli script seguenti sulle istanze di AEM Publish. Utilizza un browser web per tentare di aprire le pagine seguenti come farebbe un visitatore del sito e verifica che venga restituito il codice 404. Se ottiene qualunque altro risultato, regola i filtri.

Dovresti visualizzare il rendering normale della pagina per `/content/add_valid_page.html?debug=layout`.

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

Per determinare se l&#39;accesso in scrittura anonima è abilitato, eseguire il comando seguente in un terminale o in un prompt dei comandi. Non dovresti poter scrivere dati sul nodo.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Per tentare di annullare la validità della cache del Dispatcher e assicurarsi di ricevere una risposta del codice 403, esegui il seguente comando in un terminale o in un prompt dei comandi:

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
>Se il rendering è un&#39;istanza di AEM, è necessario installare il [Pacchetto VanityURLS-Components da Distribuzione software](https://experience.adobe.com/#/downloads/content/software-distribution/it/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) per abilitare il servizio URL personalizzato. (Per ulteriori informazioni, vedi [Distribuzione di software](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=it#software-distribution)).

Utilizza la procedura seguente per abilitare l’accesso agli URL personalizzati.

1. Se il servizio di rendering è un’istanza di AEM, installa il pacchetto `com.adobe.granite.dispatcher.vanityurl.content` sull’istanza Publish (vedi la nota precedente).
1. Per ogni URL personalizzato configurato per una pagina AEM o CQ, accertati che la configurazione [`/filter`](#configuring-access-to-content-filter) neghi l’URL. Se necessario, aggiungi un filtro che nega l’URL.
1. Aggiungi la sezione `/vanity_urls` sotto `/farms`.
1. Riavvia il server web Apache.

## Inoltro delle richieste di distribuzione del contenuto (syndication): /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

Le richieste di sindacazione sono destinate solo a Dispatcher, pertanto per impostazione predefinita non vengono inviate al renderer (ad esempio, un’istanza AEM).

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
>Il server web è responsabile della distribuzione del codice di stato corretto quando viene utilizzato il file di cache del Dispatcher, per questo è importante che lo trovi anche.

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

* L&#39;URI della richiesta contiene un punto interrogativo (`?`).
   * Indica una pagina dinamica, ad esempio un risultato di ricerca che non deve essere memorizzato nella cache.
* Se manca l’estensione del file.
   * Il server web ha bisogno dell’estensione per determinare il tipo di documento (tipo MIME).
* L&#39;intestazione di autenticazione è impostata (configurabile).
* Se l’istanza AEM risponde con le seguenti intestazioni:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>Dispatcher può memorizzare in cache i metodi GET o HEAD (per l’intestazione HTTP). Per ulteriori informazioni sul caching delle intestazioni di risposta, vedi la sezione [Caching delle intestazioni di risposta HTTP](#caching-http-response-headers).

Ciascun elemento della proprietà `/rules` include un modello [`glob`](#designing-patterns-for-glob-properties) e un tipo:

* Il modello `glob` viene utilizzato per la corrispondenza al percorso del documento.
* Il tipo definisce se memorizzare in cache i documenti che corrispondono al modello `glob`. Il valore può essere `allow` (per memorizzare il documento nella cache) o `deny` (per eseguire sempre il rendering del documento).

Se non disponi di pagine dinamiche (oltre alle pagine già escluse dalle regole di cui sopra), puoi configurare Dispatcher per memorizzare tutto nella cache. La sezione delle regole si presenta così:

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
>Non memorizzare nella cache i gruppi di utenti chiusi in quanto i diritti utente non vengono controllati per verificare la presenza di pagine memorizzate nella cache.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**Compressione**

Sui server web Apache puoi comprimere i documenti memorizzati nella cache. La compressione consente ad Apache di restituire il documento in formato compresso, se richiesto dal client. La compressione viene eseguita automaticamente abilitando il modulo Apache `mod_deflate`, ad esempio:

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

* Quando un file a un determinato livello viene invalidato, **tutto** `.stat` file dal docroot **a** il livello del file invalidato o del `statsfilevel` (a seconda di quale dei due valori è minore) vengono toccati.

   * Ad esempio: se si imposta la `statfileslevel` su 6 e un file viene invalidato al livello 5, quindi ogni `.stat` file da docroot a 5 sono toccati. Proseguendo con questo esempio, se un file viene invalidato al livello 7, ogni `stat` file da docroot a sei vengono toccati (dal momento che `/statfileslevel = "6"`).

Sono interessate solo le risorse **lungo il percorso** del file invalidato. Prendi in considerazione l’esempio seguente: un sito web utilizza la struttura `/content/myWebsite/xx/.`. Se imposti `statfileslevel` su 3, viene creato un file `.stat` come segue:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

Quando un file in `/content/myWebsite/xx` viene invalidato, quindi ogni `.stat` file da docroot verso il basso a `/content/myWebsite/xx`è commosso. Questo scenario vale solo per `/content/myWebsite/xx` e non per esempio `/content/myWebsite/yy` o `/content/anotherWebSite`.

>[!NOTE]
>
>L’annullamento della validità può essere impedito inviando un’altra intestazione `CQ-Action-Scope:ResourceOnly`. Questo metodo può essere utilizzato per svuotare determinate risorse senza invalidare altre parti della cache. Per ulteriori informazioni, vedi [questa pagina](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) e [Annullamento manuale della validità della cache di Dispatcher](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=it#configuring).

>[!NOTE]
>
>Se si specifica un valore per la proprietà `/statfileslevel`, la proprietà `/statfile` viene ignorata.

### Annullamento automatico della validità dei file memorizzati in cache {#automatically-invalidating-cached-files}

La proprietà `/invalidate` definisce i documenti che vengono invalidati automaticamente quando il contenuto viene aggiornato.

Con l’annullamento automatico della validità, Dispatcher non elimina i file memorizzati in cache dopo un aggiornamento del contenuto, ma ne verifica la validità quando vengono successivamente richiesti. I documenti nella cache che non vengono invalidati automaticamente rimangono nella cache fino a quando un aggiornamento del contenuto li elimina esplicitamente.

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
* Tutti gli altri file che corrispondono alla configurazione `/invalidate` non vengono eliminati immediatamente. Questi file vengono eliminati al momento della richiesta successiva. Nell’esempio `/content/wknd.html` non è soppresso; viene eliminato quando `/content/wknd.html` è richiesto.

Se offri file PDF e ZIP generati automaticamente per il download, potresti dover annullare automaticamente anche la validità di questi file. Un esempio di configurazione è il seguente:

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
* Azione - Azione di replica (ad esempio, Attiva, Disattiva)
* Ambito azione: ambito dell’azione di replica (vuoto, a meno che non venga inviata un’intestazione `CQ-Action-Scope: ResourceOnly`, per ulteriori dettagli, vedi [Annullamento della validità delle pagine nella cache da AEM](page-invalidate.md))

Questo metodo può essere utilizzato per coprire diversi casi d’uso. Ad esempio, l’annullamento della validità di altre cache specifiche per l’applicazione o la gestione di casi in cui l’URL esternalizzato di una pagina e la sua posizione nel docroot non corrispondono al percorso del contenuto.

Di seguito è riportato un esempio di script che registra in un file ogni richiesta di annullamento della validità.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### Esempio di script del gestore di invalidazione {#sample-invalidation-handler-script}

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
>In caso contrario, qualsiasi client può emettere una chiamata per cancellare la cache. Se eseguito ripetutamente, può influire gravemente sulle prestazioni del sito.

### Ignorare i parametri URL {#ignoring-url-parameters}

La sezione `ignoreUrlParams` definisce quali parametri URL vengono ignorati quando si stabilisce se una pagina viene memorizzata in cache o viene distribuita dalla cache:

* Quando un URL di richiesta contiene parametri tutti ignorati, la pagina viene memorizzata in cache.
* Quando un URL di richiesta contiene uno o più parametri che non vengono ignorati, la pagina non viene memorizzata in cache.

Quando un parametro viene ignorato per una pagina, questa pagine viene memorizzata in cache la prima volta che viene richiesta. Alle successive richieste per quella pagina viene distribuita la pagina dalla cache, indipendentemente dal valore del parametro nella richiesta.

>[!NOTE]
>
>È consigliabile configurare l’impostazione `ignoreUrlParams` come per creare un elenco Consentiti. In tal modo, tutti i parametri di query verranno ignorati e solo i parametri di query noti o previsti saranno esenti dall’essere ignorati. Per ulteriori dettagli ed esempi, vedi [questa pagina](https://github.com/adobe/aem-dispatcher-optimizer-tool/blob/main/docs/Rules.md#dot---the-dispatcher-publish-farm-cache-should-have-its-ignoreurlparams-rules-configured-in-an-allow-list-manner).

Per specificare quali parametri ignorare, aggiungi regole glob alla proprietà `ignoreUrlParams`:

* Per memorizzare in cache una pagina nonostante la richiesta contenente un parametro URL, crea una proprietà glob che consenta al parametro (di essere ignorato).
* Per evitare che la pagina venga memorizzata nella cache, crea una proprietà glob di tipo “deny” per impedire al parametro di essere ignorato.

>[!NOTE]
>
>Quando configuri la proprietà glob, deve corrispondere al nome del parametro query. Ad esempio, per ignorare il parametro “p1” dall’URL `http://example.com/path/test.html?p1=test&p2=v2`, la proprietà glob sarà configurata così:
> `/0002 { /glob "p1" /type "allow" }`

Il codice di esempio seguente fa sì che Dispatcher ignori tutti i parametri, tranne il parametro `nocache`. Di conseguenza, richiedi gli URL che includono `nocache` I parametri non vengono mai memorizzati nella cache da Dispatcher:

```xml
/ignoreUrlParams
{
    # allow-the-url-parameter-nocache-to-bypass-dispatcher-on-every-request
    /0001 { /glob "nocache" /type "deny" }
    # all-other-url-parameters-are-ignored-by-dispatcher-and-requests-are-cached
    /0002 { /glob "*" /type "allow" }
}
```

Nell’esempio della configurazione di `ignoreUrlParams` precedente, la seguente richiesta HTTP causa la memorizzazione nella cache della pagina perché il parametro `willbecached` viene ignorato:

```xml
GET /mypage.html?willbecached=true
```

Nell’esempio della configurazione di `ignoreUrlParams`, la seguente richiesta HTTP fa in modo che la pagina **non** venga memorizzata nella cache, perché il parametro `nocache` non viene ignorato:

```xml
GET /mypage.html?nocache=true
GET /mypage.html?nocache=true&willbecached=true
```

Per informazioni sulle proprietà glob, vedi [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

### Caching delle intestazioni di risposta HTTP {#caching-http-response-headers}

>[!NOTE]
>
>Questa funzione è disponibile con versione **4.1.11** del Dispatcher.

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
>I caratteri globbing del file non sono consentiti. Per ulteriori dettagli, vedi [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Se hai bisogno che Dispatcher memorizzi e distribuisca le intestazioni di risposta ETag da AEM, procedi come segue:
>
>* Aggiungi il nome dell’intestazione nella sezione `/cache/headers`.
>* Aggiungi quanto segue [Direttiva Apache](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) nella sezione relativa a Dispatcher:
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

Il valore predefinito è `0755` che consente al proprietario di leggere, scrivere o cercare e al gruppo e ad altri di leggere o cercare.

### Limitazione del tocco del file .stat {#throttling-stat-file-touching}

Con la proprietà `/invalidate` predefinita, ogni attivazione invalida efficacemente tutti i file `.html` (quando il loro percorso corrisponde alla sezione `/invalidate`). Su un sito web con traffico considerevole, più attivazioni successive aumentano il carico di cpu sul backend. In questo scenario, è auspicabile &quot;strozzare&quot; `.stat` touch dei file per mantenere dinamico il sito web. Puoi eseguire questa azione utilizzando il `/gracePeriod` proprietà.

La `/gracePeriod` la proprietà definisce il numero di secondi in cui una risorsa obsoleta e auto-invalidata può ancora essere servita dalla cache dopo l&#39;ultima attivazione che si è verificata. La proprietà può essere utilizzata in una configurazione in cui, altrimenti, un batch di attivazioni annullerebbe ripetutamente l’intera cache. Il valore consigliato è 2 secondi.

Per ulteriori informazioni, vedi anche le precedenti sezioni `/invalidate` e `/statfileslevel`.

### Configurazione dell’annullamento della validità della cache basata sul tempo: /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

L’annullamento della validità della cache basato su tempo dipende dal `/enableTTL` e la presenza di intestazioni di scadenza regolari dallo standard HTTP. Se imposti la proprietà su 1 (`/enableTTL "1"`), valuta le intestazioni di risposta dal backend. Se le intestazioni contengono un `Cache-Control`, `max-age` o `Expires` data, viene creato un file ausiliario vuoto accanto al file memorizzato nella cache, con l’ora di modifica uguale alla data di scadenza. Quando il file memorizzato nella cache viene richiesto oltre il tempo di modifica, viene automaticamente richiesto nuovamente dal backend.

Prima di Dispatcher 4.3.5, la logica di invalidazione TTL era basata solo sul valore TTL configurato. Con Dispatcher 4.3.5, entrambe le impostazioni TTL **e** vengono prese in considerazione le regole di invalidazione della cache del dispatcher. Di conseguenza, per un file memorizzato in cache:

1. Se la proprietà `/enableTTL` è impostata su 1, viene controllata la scadenza del file. Se il file è scaduto in base alla proprietà TTL impostata, non vengono eseguiti altri controlli e il file memorizzato in cache viene richiesto nuovamente dal back-end.
2. Se il file non è scaduto, oppure `/enableTTL` non è configurato, vengono applicate le regole standard di invalidazione della cache, ad esempio quelle impostate da [/statfileslevel](#invalidating-files-by-folder-level) e [/invalidate](#automatically-invalidating-cached-files). Questo flusso consente a Dispatcher di annullare la validità dei file per i quali il TTL non è scaduto.

Questa nuova implementazione supporta i casi di utilizzo in cui i file hanno un TTL più lungo (ad esempio, sulla rete CDN) ma possono ancora essere invalidati anche se il valore TTL non è scaduto. Favorisce la freschezza dei contenuti rispetto al rapporto hit della cache sul Dispatcher.

Viceversa, nel caso in cui sia necessario **only** la logica di scadenza applicata a un file e quindi impostata `/enableTTL` a 1 ed escludere il file dal meccanismo standard di invalidazione della cache. Ad esempio:

* Per ignorare il file, configura il [regole di invalidazione](#automatically-invalidating-cached-files) nella sezione cache. Nello snippet seguente, tutti i file che terminano con `.example.html` vengono ignorati e scadono solo quando il set TTL è passato.

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
   /0002  { /glob "*.example.html" /type "deny" }
  }
```

* Progetta la struttura del contenuto in modo da poter impostare un valore [/statfilelevel](#invalidating-files-by-folder-level) elevato affinché la validità del file non venga annullata automaticamente.

In questo modo si garantisce che `.stat` l’annullamento della validità dei file non viene utilizzato e per i file specificati è attiva solo la scadenza TTL.

>[!NOTE]
>
>Tieni presente questa impostazione `/enableTTL` a 1 abilita la memorizzazione in cache TTL solo sul lato dispatcher. Di conseguenza, le informazioni TTL contenute nel file aggiuntivo (vedi sopra) non vengono fornite ad alcun altro agente utente che richiede tale tipo di file dal dispatcher. Se desideri fornire intestazioni di memorizzazione in cache a sistemi downstream come una CDN o un browser, devi configurare la `/cache/headers` di conseguenza.

>[!NOTE]
>
>Questa funzione è disponibile nella versione **4.1.11** o successive di Dispatcher.

## Configurazione del bilanciamento del carico: /statistics {#configuring-load-balancing-statistics}

La sezione `/statistics` definisce categorie di file per i quali Dispatcher valuta la reattività di ciascun rendering. Dispatcher utilizza i punteggi per determinare a quale rendering inviare una richiesta.

Ogni categoria creata definisce un modello glob. Dispatcher confronta l’URI del contenuto richiesto con questi modelli per determinare la categoria del contenuto richiesto:

* L’ordine delle categorie determina l’ordine in cui vengono confrontate con l’URI.
* Il primo modello di categoria che corrisponde all’URI è la categoria del file. Non vengono valutati altri modelli di categoria.

Dispatcher supporta un massimo di otto categorie di statistiche. Se definisci più di otto categorie, vengono utilizzate solo le prime 8.

**Selezione rendering**

Ogni volta che Dispatcher richiede una pagina sottoposta a rendering, utilizza il seguente algoritmo per selezionare il rendering:

1. Se la richiesta contiene il nome del rendering in un cookie `renderid`, Dispatcher lo utilizza.
1. Se la richiesta non include un cookie `renderid`, Dispatcher confronta le statistiche dei rendering:

   1. Il Dispatcher determina la categoria dell’URI della richiesta.
   1. Dispatcher determina quale rendering ha il punteggio di risposta più basso per quella categoria e lo seleziona.

1. Se non è stato ancora selezionato un rendering, utilizza il primo rendering dell’elenco.

Il punteggio per la categoria di un rendering si basa sui tempi di risposta precedenti e sulle precedenti connessioni non riuscite e riuscite tentate da Dispatcher. Per ogni tentativo, viene aggiornato il punteggio per la categoria dell’URI richiesto.

>[!NOTE]
>
>Se non utilizzi il bilanciamento del carico, puoi saltare questa sezione.

### Definizione delle categorie di statistiche {#defining-statistics-categories}

Definire una categoria per ciascun tipo di documento del quale vuoi conservare le statistiche per la selezione del rendering. La sezione `/statistics` contiene una sezione `/categories`. Per definire una categoria, aggiungi una riga sotto la sezione `/categories` con il seguente formato:

`/name { /glob "pattern"}`

Il nome della categoria (`name`) deve essere univoco per la farm. La sezione `pattern` è descritta nella sezione [Progettazione di modelli per le proprietà glob](#designing-patterns-for-glob-properties).

Per determinare la categoria di un URI, Dispatcher lo confronta con ciascun modello di categoria fino a quando non viene trovata una corrispondenza. Dispatcher inizia con la prima categoria nell’elenco e continua in ordine. Quindi, inserisci prima le categorie con i modelli più specifici.

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

La `/stickyConnectionsFor` definisce una cartella contenente documenti permanenti. Questa proprietà è accessibile tramite l’URL . Dispatcher invia tutte le richieste, da un singolo utente che si trova in questa cartella, alla stessa istanza di rendering. Le connessioni permanenti garantiscono la presenza e la coerenza dei dati della sessione per tutti i documenti. Questo meccanismo utilizza il cookie `renderid`.

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

Quando le connessioni permanenti sono abilitate, il modulo Dispatcher imposta la `renderid` cookie. Questo cookie non ha il `httponly` Flag , che deve essere aggiunto per migliorare la sicurezza. Aggiungi il `httponly` impostando il flag `httpOnly` nella `/stickyConnections` nodo di un `dispatcher.any` file di configurazione. Il valore della proprietà (`0` oppure `1`) definisce se al cookie `renderid` viene aggiunto l’attributo `HttpOnly`. Il valore predefinito è `0`, il che significa che l’attributo non viene aggiunto.

Per ulteriori informazioni sul flag `httponly`, leggi [questa pagina](https://www.owasp.org/index.php/HttpOnly).

### Flag secure {#secure}

Quando le connessioni permanenti sono abilitate, il modulo Dispatcher imposta la `renderid` cookie. Questo cookie non ha il `secure` Flag , che deve essere aggiunto per migliorare la sicurezza. Aggiungi il `secure` impostazione del flag `secure` nella `/stickyConnections` nodo di un `dispatcher.any` file di configurazione. Il valore della proprietà (`0` oppure `1`) definisce se al cookie `renderid` viene aggiunto l’attributo `secure`. Il valore predefinito è `0`, il che significa che l’attributo è aggiunto **if** la richiesta in arrivo è sicura. Se il valore è impostato su `1`, il flag Secure viene aggiunto indipendentemente dal fatto che la richiesta in arrivo sia protetta o meno.

## Gestione degli errori di connessione del rendering {#handling-render-connection-errors}

Configura il comportamento di Dispatcher quando il server di rendering restituisce l’errore 500 o non è disponibile.

### Specifica di una pagina di verifica dello stato di integrità {#specifying-a-health-check-page}

Utilizzare la proprietà `/health_check` per specificare un URL che viene verificato quando viene restituito il codice di stato 500. Se questa pagina restituisce anche un codice di stato 500, l’istanza è considerata non disponibile e viene applicata una penalità di tempo configurabile ( `/unavailablePenalty`) viene applicata al rendering prima di riprovare.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifica del ritardo prima di un nuovo tentativo di richiedere una pagina {#specifying-the-page-retry-delay}

La proprietà `/retryDelay` imposta il tempo (in secondi) che Dispatcher deve attendere prima di ogni successivo turno di tentativi di connessione con i rendering nella farm. Per ogni turno, il numero massimo di tentativi di Dispatcher di connettersi a un rendering è pari al numero di rendering nella farm.

Dispatcher utilizza il valore `"1"`, se `/retryDelay` non è definito in modo esplicito. Il valore predefinito è in genere appropriato.

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

Per inviare nuovamente le richieste a render diversi quando la richiesta originale non riesce, abilita il meccanismo di failover nella farm di Dispatcher. Quando il failover è abilitato, Dispatcher si comporta come segue:

* Quando una richiesta inviata a un rendering restituisce lo stato HTTP 503 (NON DISPONIBILE), Dispatcher invia la richiesta a un altro rendering.
* Quando una richiesta inviata a un rendering restituisce lo stato HTTP 50x (diverso da 503), Dispatcher invia una richiesta per la pagina configurata per la proprietà `health_check`.
   * Se la verifica dello stato di integrità restituisce il codice 500 (INTERNAL_SERVER_ERROR), Dispatcher invia la richiesta originale a un altro rendering.
   * Se il controllo di integrità restituisce lo stato HTTP 200, Dispatcher restituisce l’errore HTTP 500 iniziale al client.

Per abilitare il failover, aggiungi la seguente riga alla farm (o al sito web):

```xml
/failover "1"
```

>[!NOTE]
>
>Per ritentare le richieste HTTP che contengono un corpo, Dispatcher invia un’intestazione di richiesta `Expect: 100-continue` al rendering prima di eseguire lo spooling del contenuto effettivo. CQ 5.5 con CQSE risponde immediatamente con il codice di stato 100 (CONTINUA) o con un codice di errore. Sono supportati anche altri contenitori servlet.

## Ignorare gli errori di interruzione: /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Questa opzione non è necessaria. Utilizzalo solo quando vedi i seguenti messaggi di log:
>
>`Error while reading response: Interrupted system call`

È possibile interrompere qualsiasi chiamata di sistema orientata al file system `EINTR` se l&#39;oggetto della chiamata di sistema si trova su un sistema remoto a cui si accede tramite NFS. Se queste chiamate di sistema possono andare in timeout o essere interrotte dipende da come il file system sottostante è stato installato sul computer locale.

Usa il parametro `/ignoreEINTR`, se la tua istanza ha questa configurazione e il registro contiene il seguente messaggio:

`Error while reading response: Interrupted system call`

Internamente, Dispatcher legge la risposta dal server remoto (ovvero, AEM) utilizzando un loop che può essere rappresentato come:

```text
while (response not finished) {  
read more data  
}
```

Questi messaggi possono essere generati quando la proprietà `EINTR` è inserita nella sezione “`read more data`” e sono dovuti alla ricezione di un segnale prima della ricezione di dati.

Per ignorare tali interruzioni, puoi aggiungere il seguente parametro a `dispatcher.any` (prima del `/farms`):

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
| `-` | Indica un intervallo di caratteri. Da utilizzare nelle classi di caratteri. Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[m-p]men.html*` Corrisponde alla seguente richiesta HTTP: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
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

Se utilizzi un **Apache** server web, puoi utilizzare la funzionalità standard per i log ruotati, i log piping o entrambi. Ad esempio, effettuando il piping dei registri:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Questa funzionalità ruota automaticamente:

* il file di registro di Dispatcher, con una marca temporale nell’estensione (`logs/dispatcher.log%Y%m%d`).
* Su base settimanale (60 x 60 x 24 x 7 = 604800 secondi).

Vedi la documentazione del server web Apache su Log Rotation e Piped Logs. Ad esempio: [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Dopo l’installazione, il livello di registro predefinito è alto (ovvero livello 3 = Debug), in modo che Dispatcher registri tutti gli errori e gli avvisi. Questo livello è utile nelle fasi iniziali.
>
>Tuttavia, tale livello richiede risorse aggiuntive. Quando Dispatcher funziona senza problemi *in base alle tue esigenze*, è possibile abbassare il livello del registro.

### Registrazione della traccia {#trace-logging}

Tra gli altri miglioramenti per Dispatcher, la versione 4.2.0 introduce anche la registrazione della traccia.

Questa funzionalità è di livello superiore alla registrazione di debug che mostra informazioni aggiuntive nei registri. La registrazione ora include:

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

1. Avvia il server web. In questo modo viene avviato anche Dispatcher.
1. Avvia l’istanza di AEM.
1. Verifica i file di registro e di errore per il server web e Dispatcher.
   * A seconda del server web, dovresti visualizzare messaggi quali:
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` e
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Naviga nel sito web tramite il server web. Verifica che il contenuto sia visualizzato nel modo corretto.\
   Ad esempio, in un’installazione locale in cui AEM viene eseguito sulla porta `4502` e il server Web sulla porta `80`, accedi alla console dei siti web utilizzando entrambi:
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * Il risultato dovrebbe essere identico. Verifica l’accesso ad altre pagine con lo stesso meccanismo.

1. Verifica che la directory della cache sia stata riempita.
1. Per verificare che la cache venga scaricata correttamente, attiva una pagina.
1. Se tutto funziona correttamente, è possibile ridurre le `loglevel` a `0`.

## Utilizzo di più istanze di Dispatcher {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più istanze di Dispatcher. Ad esempio, puoi utilizzare:

* Un’istanza di Dispatcher per pubblicare un sito web nella Intranet.
* Una seconda istanza di Dispatcher, con indirizzo e impostazioni di sicurezza diversi, per pubblicare lo stesso contenuto in Internet.

In questo caso, accertati che ogni richiesta venga gestita tramite un’unica istanza di Dispatcher. Un’istanza di Dispatcher non gestisce le richieste provenienti da un’altra istanza di Dispatcher. Accertati pertanto che entrambe le istanze di Dispatcher accedano direttamente al sito web di AEM.

## Debugging {#debugging}

Quando si aggiunge l’intestazione `X-Dispatcher-Info` a una richiesta, Dispatcher risponde se la destinazione è stata memorizzata nella cache, restituita dalla cache o non è affatto memorizzabile nella cache. L’intestazione di risposta `X-Cache-Info` contiene queste informazioni in un formato leggibile. Puoi utilizzare queste intestazioni di risposta per eseguire il debug dei problemi relativi alle risposte memorizzate in cache da Dispatcher.

Questa funzionalità non è abilitata per impostazione predefinita, quindi per l’intestazione della risposta `X-Cache-Info` per essere incluso, l&#39;azienda deve contenere la seguente voce:

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

Inoltre, il `X-Dispatcher-Info` l&#39;intestazione non richiede un valore, ma se utilizzi `curl` per il test, devi fornire un valore da inviare all’intestazione, ad esempio:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

Di seguito è riportato un elenco contenente le intestazioni di risposta che `X-Dispatcher-Info` restituisce:

* **memorizzato in cache**\
   Il file di destinazione è contenuto nella cache e Dispatcher ha determinato che è valido per consegnarlo.
* **caching**\
   Il file di destinazione non è contenuto nella cache e Dispatcher ha determinato che è valido memorizzare nella cache l’output e consegnarlo.
* **caching: il file stat è più recente**
Il file di destinazione è contenuto nella cache, tuttavia, viene invalidato da un file stat più recente. Il Dispatcher elimina il file di destinazione, lo ricrea dall’output e lo distribuisce.
* **non memorizzabile in cache: nessuna directory principale dei documenti**
La configurazione della farm non contiene una directory principale dei documenti (elemento di configurazione 
`cache.docroot`).
* **non memorizzabile in cache: percorso del file della cache troppo lungo**\
   Il file di destinazione, ovvero la concatenazione di directory principale del documento e file URL, supera la lunghezza massima consentita per i nomi di file sul sistema.
* **non memorizzabile in cache: percorso del file temporaneo troppo lungo**\
   Il modello di nome file temporaneo supera la lunghezza massima consentita per i nomi di file sul sistema. Dispatcher crea prima un file temporaneo prima di creare o sovrascrivere effettivamente il file memorizzato nella cache. Il nome del file temporaneo è il nome del file di destinazione con i caratteri `_YYYYXXXXXX` aggiunto, dove `Y` e `X` vengono sostituiti per creare un nome univoco.
* **non memorizzabile in cache: l’URL della richiesta non ha estensione**\
   L’URL della richiesta non ha un’estensione oppure un percorso segue l’estensione del file, ad esempio: `/test.html/a/path`.
* **non memorizzabile in cache: non era una richiesta di GET o HEAD**
Il metodo HTTP non è un GET o un HEAD. Dispatcher presuppone che l’output contenga dati dinamici che non devono essere memorizzati nella cache.
* **non memorizzabile in cache: la richiesta conteneva una stringa di query**\
   La richiesta conteneva una stringa di query. Dispatcher presuppone che l’output dipenda dalla stringa di query specificata e quindi non memorizza in cache.
* **non memorizzabile in cache: il gestore di sessione non ha autenticato**\
   La cache della farm è gestita da un gestore di sessione (la configurazione contiene un nodo `sessionmanagement`) e la richiesta non conteneva le informazioni di autenticazione appropriate.
* **non memorizzabile in cache: la richiesta contiene un’autorizzazione**\
   La farm non può memorizzare in cache l’output (`allowAuthorized 0`) e la richiesta contiene informazioni di autenticazione.
* **non memorizzabile in cache: la destinazione è una directory**\
   Il file di destinazione è una directory. Questa posizione potrebbe indicare un errore concettuale, in cui un URL e alcuni URL secondari contengono entrambi un output memorizzabile nella cache. Ad esempio, se una richiesta a `/test.html/a/file.ext` viene prima e contiene l’output in cache, Dispatcher non è in grado di memorizzare nella cache l’output di una richiesta successiva a `/test.html`.
* **non memorizzabile in cache: l’URL della richiesta ha una barra finale**\
   L’URL della richiesta ha una barra finale.
* **non memorizzabile in cache: URL della richiesta non incluso nelle regole di cache**\
   Le regole di cache della farm negano esplicitamente il caching dell’output di alcuni URL di richiesta.
* **non memorizzabile in cache: accesso negato dal modulo AuthChecker**\
   Il modulo AuthChecker della farm ha negato l’accesso al file memorizzato in cache.
* **non memorizzabile in cache: sessione non valida**
La cache della farm è gestita da un gestore di sessione (la configurazione contiene un nodo `sessionmanagement`) e la sessione dell’utente non è o non è più valida.
* **non memorizzabile in cache: la risposta contiene`no_cache`**
Il server remoto ha restituito un’intestazione 
`Dispatcher: no_cache` intestazione, impossibile memorizzare in cache l’output in Dispatcher.
* **non memorizzabile in cache: la lunghezza del contenuto della risposta è zero**
La lunghezza del contenuto della risposta è zero; Dispatcher non crea un file di lunghezza zero.
