---
title: Configurazione di Dispatcher
description: Scopri come configurare Dispatcher.
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: 35739785aa835a0b995fab4710a0e37bd0ff62b4
workflow-type: tm+mt
source-wordcount: '8512'
ht-degree: 2%

---

# Configurazione di Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

Le sezioni seguenti descrivono come configurare vari aspetti del Dispatcher.

## Supporto per IPv4 e IPv6 {#support-for-ipv-and-ipv}

Tutti gli elementi di AEM e Dispatcher possono essere installati nelle reti IPv4 e IPv6. Vedere [IPV4 e IPV6](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=en#ipv-and-ipv).

## File di configurazione del dispatcher {#dispatcher-configuration-files}

Per impostazione predefinita, la configurazione del Dispatcher viene memorizzata nel file di testo `dispatcher.any`, anche se è possibile modificare il nome e il percorso del file durante l&#39;installazione.

Il file di configurazione contiene una serie di proprietà con un singolo valore o con più valori che controllano il comportamento di Dispatcher:

* I nomi delle proprietà hanno il prefisso &quot;forward slash `/`&quot;.
* Le proprietà con più valori racchiudono gli elementi secondari utilizzando le parentesi graffe `{ }`.

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

Puoi includere altri file che contribuiscono alla configurazione:

* Se il file di configurazione è grande, puoi suddividerlo in diversi file più piccoli (che sono più facili da gestire) e includerli.
* Per includere i file generati automaticamente.

Ad esempio, per includere il file myFarm.any nella configurazione /farms, utilizza il seguente codice:

```xml
/farms
  {
  $include "myFarm.any"
  }
```

Utilizza l&#39;asterisco (`*`) come carattere jolly per specificare un intervallo di file da includere.

Ad esempio, se i file da `farm_1.any` a `farm_5.any` contengono la configurazione delle farm da uno a cinque, puoi includerli come segue:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Utilizzo delle variabili di ambiente {#using-environment-variables}

Puoi utilizzare le variabili di ambiente nelle proprietà con valori stringa nel file dispatcher.any invece di codificare i valori. Per includere il valore di una variabile di ambiente, utilizza il formato `${variable_name}`.

Ad esempio, se il file dispatcher.any si trova nella stessa directory della directory cache, è possibile utilizzare il seguente valore per la proprietà [docroot](#specifying-the-cache-directory) :

```xml
/docroot "${PWD}/cache"
```

Come altro esempio, se crei una variabile di ambiente denominata `PUBLISH_IP` che memorizza il nome host dell&#39;istanza di pubblicazione AEM, puoi utilizzare la seguente configurazione della proprietà [/renders](#defining-page-renderers-renders) :

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Denominazione dell’istanza di Dispatcher {#naming-the-dispatcher-instance-name}

Utilizza la proprietà `/name` per specificare un nome univoco per identificare l’istanza di Dispatcher. La proprietà `/name` è una proprietà di livello principale nella struttura di configurazione.

## Definizione delle aziende {#defining-farms-farms}

La proprietà `/farms` definisce uno o più set di comportamenti del Dispatcher, in cui ogni set è associato a siti web o URL diversi. La proprietà `/farms` può includere una o più farm:

* Utilizza una singola farm quando desideri che Dispatcher gestisca allo stesso modo tutte le pagine web o i siti web.
* Crea più farm quando diverse aree del sito web o siti web diversi richiedono un comportamento diverso da Dispatcher.

La proprietà `/farms` è una proprietà di livello principale nella struttura di configurazione. Per definire una farm, aggiungi una proprietà figlio alla proprietà `/farms` . Utilizza un nome di proprietà che identifica in modo univoco la farm all’interno dell’istanza di Dispatcher.

La proprietà `/farmname` ha più valori e contiene altre proprietà che definiscono il comportamento di Dispatcher:

* Gli URL delle pagine a cui si applica la farm.
* Uno o più URL di servizio (in genere AEM istanze di pubblicazione) da utilizzare per il rendering dei documenti.
* Statistiche da utilizzare per il bilanciamento del carico tra più render di documenti.
* Diversi altri comportamenti, ad esempio quali file memorizzare in cache e dove.

Il valore può includere qualsiasi carattere alfanumerico (a-z, 0-9). L’esempio seguente mostra la definizione dello scheletro per due farm denominate `/daycom` e `/docsdaycom`:

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
>Se utilizzi più di una farm di rendering, l’elenco viene valutato dal basso verso l’alto. Questo è particolarmente importante quando definisci [Host virtuali](#identifying-virtual-hosts-virtualhosts) per i siti web.

Ogni proprietà farm può contenere le seguenti proprietà figlio:

| Nome proprietà | Descrizione |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | Home page predefinita (opzionale)(solo IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | Le intestazioni della richiesta HTTP client da trasmettere. |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | Host virtuali per la farm. |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | Supporto per la gestione e l’autenticazione delle sessioni. |
| [/renders](#defining-page-renderers-renders) | I server che forniscono le pagine di cui è stato eseguito il rendering (in genere AEM istanze di pubblicazione). |
| [/filter](#configuring-access-to-content-filter) | Definisce gli URL a cui Dispatcher consente l’accesso. |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | Configura l’accesso agli URL personalizzati. |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | Supporto per l&#39;inoltro delle richieste di sindacazione. |
| [/cache](#configuring-the-dispatcher-cache-cache) | Configura il comportamento di caching. |
| [/statistiche](#configuring-load-balancing-statistics) | Definizione di categorie statistiche per i calcoli di bilanciamento del carico. |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | Cartella contenente documenti permanenti. |
| [/health_check](#specifying-a-health-check-page) | URL da utilizzare per determinare la disponibilità del server. |
| [/tryDelay](#specifying-the-page-retry-delay) | Ritardo prima di provare nuovamente una connessione non riuscita. |
| [/non disponibilePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | Sanzioni che incidono sulle statistiche relative ai calcoli di bilanciamento del carico. |
| [/failover](#using-the-failover-mechanism) | Invia le richieste a rendering diversi quando la richiesta originale non riesce. |
| [/auth_checker](permissions-cache.md) | Per la memorizzazione in cache sensibile alle autorizzazioni, consulta [Memorizzazione in cache di contenuto protetto](permissions-cache.md). |

## Specificare una pagina predefinita (solo IIS) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>Il parametro `/homepage`(solo IIS) non funziona più. È invece necessario utilizzare il modulo [Riscrittura URL IIS](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>Se utilizzi Apache, utilizza il modulo `mod_rewrite` . Per informazioni su `mod_rewrite`, consulta la documentazione del sito web Apache (ad esempio, [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). Quando si utilizza `mod_rewrite`, è consigliabile utilizzare il flag **[&#39;passthrough|PT&#39; (passare al gestore successivo)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** per forzare il motore di riscrittura a impostare il campo `uri` della struttura interna `request_rec` sul valore del campo `filename`.

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

La proprietà `/clientheaders` definisce un elenco di intestazioni HTTP che Dispatcher trasmette dalla richiesta HTTP del client al renderer (istanza AEM).

Per impostazione predefinita, Dispatcher inoltra le intestazioni HTTP standard all’istanza AEM. In alcuni casi, potresti voler inoltrare intestazioni aggiuntive o rimuovere intestazioni specifiche:

* Aggiungi intestazioni, ad esempio intestazioni personalizzate, che la tua istanza AEM prevede nella richiesta HTTP.
* Rimuovere le intestazioni, ad esempio le intestazioni di autenticazione, rilevanti solo per il server web.

Se si personalizza il set di intestazioni da trasmettere, è necessario specificare un elenco completo di intestazioni, comprese quelle che sono normalmente incluse per impostazione predefinita.

Ad esempio, un’istanza di Dispatcher che gestisce le richieste di attivazione della pagina per le istanze di pubblicazione richiede l’intestazione `PATH` nella sezione `/clientheaders` . L’intestazione `PATH` abilita la comunicazione tra l’agente di replica e il dispatcher.

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

## Identificazione di host virtuali {#identifying-virtual-hosts-virtualhosts}

La proprietà `/virtualhosts` definisce un elenco di tutte le combinazioni hostname/URI accettate da Dispatcher per questa farm. È possibile utilizzare il carattere asterisco (`*`) come carattere jolly. I valori per la proprietà / `virtualhosts` utilizzano il formato seguente:

```xml
[scheme]host[uri][*]
```

* `scheme`: (Facoltativo)  `https://` o  `https://.`
* `host`: Nome o indirizzo IP del computer host e, se necessario, numero di porta. (Vedere [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`: (Facoltativo) Percorso delle risorse.

Nell&#39;esempio seguente la configurazione gestisce le richieste per i domini .com e .ch della mia azienda e per tutti i domini di mySubDivision:

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

La seguente configurazione gestisce le richieste *all* :

```xml
   /virtualhosts
    {
    "*"
    }
```

### Risoluzione dell&#39;host virtuale {#resolving-the-virtual-host}

Quando Dispatcher riceve una richiesta HTTP o HTTPS, trova il valore host virtuale che corrisponde meglio alle intestazioni `host,` `uri` e `scheme` della richiesta. Dispatcher valuta i valori delle proprietà `virtualhosts` nell’ordine seguente:

* Il Dispatcher inizia dalla farm più bassa e procede verso l’alto nel file dispatcher.any.
* Per ogni farm, Dispatcher inizia con il valore più in alto nella proprietà `virtualhosts` e procede verso il basso nell’elenco dei valori.

Dispatcher trova il valore host virtuale che corrisponde meglio al valore di host virtuale nel modo seguente:

* Viene utilizzato il primo host virtuale rilevato che corrisponde a tutti e tre i `host`, `scheme` e `uri` della richiesta.
* Se nessun valore `virtualhosts` dispone di parti `scheme` e `uri` che corrispondono sia a `scheme` che a `uri` della richiesta, viene utilizzato il primo host virtuale rilevato che corrisponde a `host` della richiesta.
* Se nessun valore `virtualhosts` ha una parte host che corrisponde all’host della richiesta, viene utilizzato l’host virtuale più in alto della farm più in alto.

Pertanto, devi posizionare l’host virtuale predefinito nella parte superiore della proprietà `virtualhosts` nella farm più in alto del file `dispatcher.any`.

### Esempio di risoluzione host virtuale {#example-virtual-host-resolution}

L’esempio seguente rappresenta uno snippet da un file `dispatcher.any` che definisce due farm del Dispatcher e ogni farm definisce una proprietà `virtualhosts` .

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

Utilizzando questo esempio, la tabella seguente mostra gli host virtuali risolti per le richieste HTTP specificate:

| URL richiesta | Host virtuale risolto |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## Abilitazione di sessioni sicure - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` **Per abilitare questa funzione, devi impostare** su  `"0"` nella  `/cache` sezione .

Crea una sessione protetta per l’accesso alla farm di rendering in modo che gli utenti debbano accedere a qualsiasi pagina della farm. Dopo l’accesso, gli utenti possono accedere alle pagine della farm. Per informazioni sull’utilizzo di questa funzione con CUG, consulta [Creazione di un gruppo utenti chiuso](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=en#creating-the-user-group-to-be-used) . Inoltre, consulta Dispatcher [Lista di controllo sicurezza](/help/using/security-checklist.md) prima di iniziare la pubblicazione.

La proprietà `/sessionmanagement` è una sottoproprietà di `/farms`.

>[!CAUTION]
>
>Se le sezioni del sito web utilizzano requisiti di accesso diversi, è necessario definire più farm.

**/** sessionmanagementhas diversi sottoparametri:

**/directory** (obbligatorio)

La directory in cui sono memorizzate le informazioni sulla sessione. Se la directory non esiste, viene creata.

>[!CAUTION]
>
> Durante la configurazione del sottoparametro della directory **non** puntare alla cartella principale (`/directory "/"`) in quanto può causare gravi problemi. È sempre necessario specificare il percorso della cartella in cui sono memorizzate le informazioni sulla sessione. Esempio:

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (facoltativo)

Codifica delle informazioni sulla sessione. Utilizza `md5` per la crittografia utilizzando l&#39;algoritmo md5 o `hex` per la codifica esadecimale. Se si crittografano i dati della sessione, un utente con accesso al file system non può leggere il contenuto della sessione. Il valore predefinito è `md5`.

**/header**  (facoltativo)

Nome dell&#39;intestazione HTTP o del cookie che memorizza le informazioni di autorizzazione. Se archivi le informazioni nell’intestazione http, utilizza `HTTP:<header-name>`. Per memorizzare le informazioni in un cookie, utilizza `Cookie:<header-name>`. Se non si specifica un valore, viene utilizzato `HTTP:authorization`.

**/timeout**  (facoltativo)

Il numero di secondi prima che la sessione si esaurisca dopo l’ultimo utilizzo. Se non viene specificato `"800"`, la sessione scade poco più di 13 minuti dopo l’ultima richiesta dell’utente.

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

La proprietà /renders definisce l’URL a cui Dispatcher invia le richieste di rendering di un documento. L&#39;esempio seguente `/renders` identifica una singola istanza AEM per il rendering:

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

La seguente sezione /renders di esempio identifica un&#39;istanza AEM che viene eseguita sullo stesso computer del dispatcher:

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

La seguente sezione di esempio /renders distribuisce le richieste di rendering equamente tra due istanze AEM:

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

Specifica il timeout di connessione per l&#39;accesso all&#39;istanza AEM in millisecondi. Il valore predefinito è `"0"` e il Dispatcher deve attendere indefinitamente.

**/receiveTimeout**

Specifica il tempo in millisecondi che una risposta può richiedere. Il valore predefinito è `"600000"` e Dispatcher deve attendere 10 minuti. Un&#39;impostazione di `"0"` elimina completamente il timeout.

Se viene raggiunto il timeout durante l&#39;analisi delle intestazioni di risposta, viene restituito uno stato HTTP pari a 504 (Bad Gateway). Se il timeout viene raggiunto durante la lettura del corpo della risposta, Dispatcher restituisce al client la risposta incompleta, ma elimina tutti i file di cache che potrebbero essere stati scritti.

**/ipv4**

Specifica se Dispatcher utilizza la funzione `getaddrinfo` (per IPv6) o la funzione `gethostbyname` (per IPv4) per ottenere l’indirizzo IP del rendering. Se si imposta il valore 0, viene utilizzato `getaddrinfo`. Se si utilizza un valore di `1`, viene utilizzato `gethostbyname`. Il valore predefinito è `0`.

La funzione `getaddrinfo` restituisce un elenco di indirizzi IP. Dispatcher esegue l’esecuzione dell’elenco degli indirizzi fino a quando non stabilisce una connessione TCP/IP. Pertanto, la proprietà `ipv4` è importante quando il nome host di rendering è associato a più indirizzi IP e l’host, in risposta alla funzione `getaddrinfo` , restituisce un elenco di indirizzi IP che sono sempre nello stesso ordine. In questa situazione, utilizza la funzione `gethostbyname` in modo che l’indirizzo IP con cui Dispatcher si connette sia randomizzato.

L’ELB (Elastic Load Balancing) di Amazon è un servizio che risponde a getaddrinfo con un elenco potenzialmente identico di indirizzi IP.

**/secure**

Se la proprietà `/secure` ha un valore di `"1"` Dispatcher utilizza HTTPS per comunicare con l’istanza AEM. Per ulteriori dettagli, consulta anche [Configurazione del Dispatcher per l’utilizzo di SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

Con la versione di Dispatcher **4.1.6**, puoi configurare la proprietà `/always-resolve` come segue:

* Se impostato su `"1"`, risolverà il nome host su ogni richiesta (Dispatcher non memorizzerà mai nella cache alcun indirizzo IP). Potrebbe esserci un leggero impatto sulle prestazioni a causa della chiamata aggiuntiva necessaria per ottenere le informazioni sull’host per ogni richiesta.
* Se la proprietà non è impostata, l’indirizzo IP verrà memorizzato nella cache per impostazione predefinita.

Inoltre, questa proprietà può essere utilizzata nel caso di problemi di risoluzione IP dinamica, come illustrato nell&#39;esempio seguente:

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

Utilizza la sezione `/filter` per specificare le richieste HTTP accettate da Dispatcher. Tutte le altre richieste vengono rimandate al server web con un codice di errore 404 (pagina non trovata). Se non esiste alcuna sezione `/filter`, tutte le richieste vengono accettate.

**Nota:** le richieste per lo  [](#naming-the-statfile) statfilevengono sempre rifiutate.

>[!CAUTION]
>
>Per ulteriori considerazioni sulla limitazione dell’accesso tramite Dispatcher, consulta la [Lista di controllo della sicurezza del dispatcher](security-checklist.md) . Inoltre, leggi la [AEM lista di controllo della sicurezza](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/security-checklist.html?lang=en#security) per ulteriori dettagli di sicurezza relativi all&#39;installazione AEM.

La sezione `/filter` consiste in una serie di regole che negano o consentono l’accesso al contenuto in base ai pattern presenti nella parte della riga della richiesta HTTP. Utilizza una strategia di elenco consentiti per la sezione `/filter`:

* In primo luogo, nega l&#39;accesso a tutto.
* Consenti l’accesso al contenuto in base alle esigenze.

### Definizione di un filtro {#defining-a-filter}

Ogni elemento della sezione `/filter` include un tipo e un pattern corrispondenti a un elemento specifico della riga della richiesta o all’intera riga della richiesta. Ogni filtro può contenere i seguenti elementi:

* **Tipo**: Il  `/type` indica se consentire o negare l’accesso per le richieste che corrispondono al pattern. Il valore può essere `allow` o `deny`.

* **Elemento della riga di richiesta:** Includi  `/method`,  `/url`,  `/query` o  `/protocol` e un pattern per filtrare le richieste in base a queste parti specifiche della parte della riga di richiesta della richiesta HTTP. Il filtro sugli elementi della riga di richiesta (anziché sull’intera riga di richiesta) è il metodo di filtro preferito.

* **Elementi avanzati della riga di richiesta:** a partire da Dispatcher 4.2.0, sono disponibili quattro nuovi elementi di filtro. Questi nuovi elementi sono rispettivamente `/path`, `/selectors`, `/extension` e `/suffix`. Includi uno o più di questi elementi per controllare ulteriormente i pattern URL.

>[!NOTE]
>
>Per ulteriori informazioni sulla parte della riga di richiesta a cui fa riferimento ciascuno di questi elementi, consulta la pagina wiki [Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) .

* **Proprietà** glob: La  `/glob` proprietà viene utilizzata per corrispondere all&#39;intera riga di richiesta della richiesta HTTP.

>[!CAUTION]
>
>In Dispatcher il filtraggio con i globs è obsoleto. Di conseguenza, evita di utilizzare i globs nelle sezioni `/filter` in quanto potrebbero causare problemi di sicurezza. Quindi, invece di:
>
>`/glob "* *.css *"`
>
>dovrebbe utilizzare
>
>`/url "*.css"`

#### Parte della riga di richiesta delle richieste HTTP {#the-request-line-part-of-http-requests}

HTTP/1.1 definisce la [riga-richiesta](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) come segue:

`Method Request-URI HTTP-Version<CRLF>`

I caratteri `<CRLF>` rappresentano un ritorno a capo seguito da un avanzamento di riga. L’esempio seguente è la riga di richiesta ricevuta quando un cliente richiede la pagina inglese US del sito WKND:

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

I pattern devono tenere conto dei caratteri di spazio nella riga della richiesta e dei caratteri `<CRLF>`.

#### Virgolette doppie e virgolette singole {#double-quotes-vs-single-quotes}

Quando crei le regole del filtro, utilizza le virgolette doppie `"pattern"` per i pattern semplici. Se utilizzi Dispatcher 4.2.0 o versione successiva e il pattern include un’espressione regolare, devi racchiudere il pattern regex `'(pattern1|pattern2)'` tra virgolette singole.

#### Espressioni regolari {#regular-expressions}

Nelle versioni di Dispatcher successive alla versione 4.2.0, è possibile includere nei pattern di filtro le espressioni regolari POSIX estese.

#### Risoluzione dei problemi dei filtri {#troubleshooting-filters}

Se i filtri non si attivano come previsto, abilita [Trace Logging](#trace-logging) sul dispatcher in modo da vedere quale filtro intercetta la richiesta.

#### Esempio di filtro: Rifiuta tutto {#example-filter-deny-all}

La seguente sezione del filtro di esempio causa il rifiuto da parte di Dispatcher di richiedere tutti i file. È necessario negare l’accesso a tutti i file e quindi consentire l’accesso a aree specifiche.

```xml
  /0001  { /glob "*" /type "deny" }
```

Le richieste a un&#39;area negata in modo esplicito causano la restituzione di un codice di errore 404 (pagina non trovata).

#### Esempio di filtro: Negare l&#39;accesso a aree specifiche {#example-filter-deny-access-to-specific-areas}

I filtri consentono inoltre di negare l’accesso a vari elementi, ad esempio pagine ASP e aree sensibili all’interno di un’istanza di pubblicazione. Il filtro seguente nega l&#39;accesso alle pagine ASP:

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Esempio di filtro: Abilitare le richieste POST {#example-filter-enable-post-requests}

Il filtro di esempio seguente consente l’invio dei dati del modulo tramite il metodo POST:

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

Se l’istanza di pubblicazione utilizza un contesto dell’applicazione web (ad esempio pubblica), questo può essere aggiunto anche alla definizione del filtro.

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

Se è comunque necessario accedere a singole pagine all’interno dell’area riservata, è possibile accedervi. Ad esempio, per consentire l’accesso alla scheda Archivio nella console Flusso di lavoro, aggiungi la seguente sezione:

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>Quando a una richiesta sono applicati più pattern di filtri, l’ultimo pattern di filtro applicato è valido.

#### Esempio di filtro: Utilizzo di espressioni regolari {#example-filter-using-regular-expressions}

Questo filtro abilita le estensioni nelle directory di contenuto non pubblico utilizzando un’espressione regolare, definita qui tra virgolette singole:

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Esempio di filtro: Filtrare elementi aggiuntivi di un URL di richiesta {#example-filter-filter-additional-elements-of-a-request-url}

Di seguito è riportato un esempio di regola che blocca l’acquisizione del contenuto dal percorso `/content` e dalla relativa struttura secondaria, utilizzando i filtri per percorso, selettori ed estensioni:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### Esempio /filter, sezione {#example-filter-section}

Quando configuri Dispatcher, devi limitare il più possibile l’accesso esterno. L’esempio seguente fornisce un accesso minimo per i visitatori esterni:

* `/content`
* contenuti vari, quali progetti e librerie client; ad esempio:

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

Dopo aver creato i filtri, [verifica l&#39;accesso alla pagina](#testing-dispatcher-security) per garantire la protezione dell&#39;istanza AEM.

La seguente sezione `/filter` del file `dispatcher.any` può essere utilizzata come base nel file di configurazione [Dispatcher.](#dispatcher-configuration-files)

Questo esempio si basa sul file di configurazione predefinito fornito con Dispatcher e deve essere utilizzato come esempio in un ambiente di produzione. Gli elementi con prefisso `#` vengono disattivati (con commento). Presta attenzione se decidi di attivarli (rimuovendo il `#` su tale riga) in quanto questo può avere un impatto sulla sicurezza.

È necessario negare l’accesso a tutto, quindi consentire l’accesso a elementi specifici (limitati):

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
>Se utilizzato con Apache, progetta i pattern dell’URL del filtro in base alla proprietà DispatcherUseProcessedURL del modulo Dispatcher. (Consulta [Apache Web Server - Configurare il server Web Apache per Dispatcher](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher).)

>[!NOTE]
>
>I filtri `0030` e `0031` relativi a Dynamic Media sono applicabili a AEM 6.0 e versioni successive.

Se scegli di estendere l’accesso, prendi in considerazione le seguenti raccomandazioni:

* L&#39;accesso esterno a `/admin` deve essere sempre *completamente* disabilitato se utilizzi CQ versione 5.4 o una versione precedente.

* Presta attenzione quando consenti l’accesso ai file in `/libs`. L&#39;accesso dovrebbe essere consentito su base individuale.
* Nega l&#39;accesso alla configurazione di replica in modo che non possa essere visualizzata:

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* Negare l&#39;accesso al proxy inverso Google Gadgets:

   * `/libs/opensocial/proxy*`

A seconda dell’installazione, potrebbero essere disponibili risorse aggiuntive in `/libs`, `/apps` o altrove. Puoi utilizzare il file `access.log` come metodo per determinare le risorse a cui si accede esternamente.

>[!CAUTION]
>
>L’accesso alle console e alle directory può rappresentare un rischio per la sicurezza degli ambienti di produzione. A meno che tu non abbia esplicite giustificazioni, devono rimanere disattivati (commentato).

>[!CAUTION]
>
>Se utilizzi [rapporti in un ambiente di pubblicazione](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=en#using-reports-in-a-publish-environment), configura Dispatcher per negare l’accesso a `/etc/reports` ai visitatori esterni.

### Limitazione delle stringhe di query {#restricting-query-strings}

A partire dalla versione 4.1.5 di Dispatcher, utilizza la sezione `/filter` per limitare le stringhe di query. Si consiglia vivamente di consentire esplicitamente le stringhe di query ed escludere la tolleranza generica tramite gli elementi di filtro `allow`.

Una singola voce può avere `glob` o una combinazione di `method`, `url`, `query` e `version`, ma non entrambe. L&#39;esempio seguente consente la stringa di query `a=*` e nega tutte le altre stringhe di query per gli URL che si risolvono nel nodo `/etc` :

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>Se una regola contiene un elemento `/query`, corrisponderà solo alle richieste che contengono una stringa di query e corrispondono al pattern di query specificato.
>
>Nell’esempio precedente, se le richieste a `/etc` prive di stringa di query devono essere consentite anche, sono necessarie le seguenti regole:


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Verifica della sicurezza del Dispatcher {#testing-dispatcher-security}

I filtri di Dispatcher devono bloccare l’accesso alle pagine e agli script seguenti su AEM istanze di pubblicazione. Utilizza un browser web per tentare di aprire le pagine seguenti come farebbe un visitatore del sito e verifica che venga restituito un codice 404. Se si ottiene un altro risultato, regola i filtri.

Tieni presente che è necessario visualizzare il rendering normale della pagina per `/content/add_valid_page.html?debug=layout`.

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

Eseguire il comando seguente in un terminale o in un prompt dei comandi per determinare se l&#39;accesso in scrittura anonima è abilitato. Non dovresti essere in grado di scrivere dati sul nodo.

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

Esegui il seguente comando in un terminale o in un prompt dei comandi per tentare di annullare la validità della cache del Dispatcher e assicurati di ricevere una risposta codice 404:

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

Quando l’accesso agli URL personalizzati è abilitato, Dispatcher chiama periodicamente un servizio che viene eseguito sull’istanza di rendering per ottenere un elenco di URL personalizzati. Dispatcher memorizza l’elenco in un file locale. Quando una richiesta di pagina viene negata a causa di un filtro nella sezione `/filter`, Dispatcher consulta l’elenco degli URL personalizzati. Se l’URL negato è presente nell’elenco, Dispatcher consente l’accesso all’URL personalizzato.

Per abilitare l’accesso agli URL personalizzati, aggiungi una sezione `/vanity_urls` alla sezione `/farms`, simile all’esempio seguente:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

La sezione `/vanity_urls` contiene le seguenti proprietà:

* `/url`: Percorso del servizio URL personalizzato in esecuzione sull&#39;istanza di rendering. Il valore di questa proprietà deve essere `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`: Percorso del file locale in cui Dispatcher memorizza l’elenco degli URL personalizzati. Assicurati che Dispatcher abbia accesso in scrittura a questo file.
* `/delay`: (Secondi) Il tempo tra le chiamate al servizio URL personalizzato.

>[!NOTE]
>
>Se il rendering è un&#39;istanza di AEM, è necessario installare il pacchetto [VanityURLS-Components da Software Distribution](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) per abilitare il servizio URL personalizzato. (Per ulteriori informazioni, consulta [Distribuzione di software](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=en#software-distribution) .)

Segui la procedura seguente per abilitare l’accesso agli URL personalizzati.

1. Se il servizio di rendering è un’istanza AEM, installa il pacchetto `com.adobe.granite.dispatcher.vanityurl.content` sull’istanza di pubblicazione (vedi la nota precedente).
1. Per ogni URL personalizzato configurato per una pagina AEM o CQ, accertati che la configurazione [`/filter`](#configuring-access-to-content-filter) neghi l’URL. Se necessario, aggiungi un filtro che nega l’URL.
1. Aggiungi la sezione `/vanity_urls` sotto `/farms`.
1. Riavvia il server web Apache.

## Inoltro delle richieste di sindacazione - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

Le richieste di sindacazione sono solitamente destinate solo a Dispatcher, pertanto per impostazione predefinita non vengono inviate al renderer (ad esempio, un’istanza AEM).

Se necessario, imposta la proprietà `/propagateSyndPost` su `"1"` per inoltrare le richieste di sindacazione a Dispatcher. Se impostato, assicurati che le richieste di POST non siano negate nella sezione del filtro .

## Configurazione della cache del Dispatcher - /cache {#configuring-the-dispatcher-cache-cache}

La sezione `/cache` controlla come Dispatcher memorizza nella cache i documenti. Configura diverse sottoproprietà per implementare le strategie di caching:

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

Una sezione di cache di esempio potrebbe essere la seguente:

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
>Per la memorizzazione in cache sensibile alle autorizzazioni, leggere [Memorizzazione in cache di contenuto protetto](permissions-cache.md).

### Specifica della directory cache {#specifying-the-cache-directory}

La proprietà `/docroot` identifica la directory in cui vengono memorizzati i file memorizzati nella cache.

>[!NOTE]
>
>Il valore deve corrispondere esattamente al percorso della directory principale del documento del server Web in modo che Dispatcher e il server Web gestiscano gli stessi file.\
>Il server web è responsabile della distribuzione del codice di stato corretto quando viene utilizzato il file di cache del dispatcher, per questo è importante che possa trovarlo.

Se si utilizzano più farm, ogni farm deve utilizzare una radice documento diversa.

### Denominazione dello statfile {#naming-the-statfile}

La proprietà `/statfile` identifica il file da utilizzare come file di stato. Dispatcher utilizza questo file per registrare l’ora dell’aggiornamento del contenuto più recente. Lo statfile può essere qualsiasi file sul server web.

Lo statfile non ha contenuto. Quando il contenuto viene aggiornato, Dispatcher aggiorna la marca temporale. Lo statfile predefinito è denominato `.stat` e viene memorizzato nel docroot. Dispatcher blocca l’accesso allo statfile.

>[!NOTE]
>
>Se `/statfileslevel` è configurato, Dispatcher ignora la proprietà `/statfile` e utilizza `.stat` come nome.

### Servizio di documenti obsoleti in caso di errori {#serving-stale-documents-when-errors-occur}

La proprietà `/serveStaleOnError` controlla se Dispatcher restituisce documenti invalidati quando il server di rendering restituisce un errore. Per impostazione predefinita, quando uno statfile viene toccato e invalida il contenuto nella cache, Dispatcher lo elimina alla successiva richiesta.

Se `/serveStaleOnError` è impostato su `"1"`, Dispatcher non elimina il contenuto invalidato dalla cache a meno che il server di rendering non restituisca una risposta corretta. Una risposta 5xx da AEM o un timeout di connessione fa sì che Dispatcher distribuisca il contenuto obsoleto e risponda con e lo stato HTTP 111 (Revalidation Failed).

### Memorizzazione in cache quando viene utilizzata l&#39;autenticazione {#caching-when-authentication-is-used}

La proprietà `/allowAuthorized` controlla se le richieste contenenti una delle seguenti informazioni di autenticazione sono memorizzate nella cache:

* Intestazione `authorization`
* Un cookie denominato `authorization`
* Un cookie denominato `login-token`

Per impostazione predefinita, le richieste che includono queste informazioni di autenticazione non vengono memorizzate nella cache perché l’autenticazione non viene eseguita quando un documento memorizzato nella cache viene restituito al client. Questa configurazione impedisce a Dispatcher di servire i documenti memorizzati nella cache a utenti che non dispongono dei diritti necessari.

Tuttavia, se i requisiti consentono la memorizzazione in cache dei documenti autenticati, impostare `/allowAuthorized` su uno:

`/allowAuthorized "1"`

>[!NOTE]
>
>Per abilitare la gestione delle sessioni (utilizzando la proprietà `/sessionmanagement` ), la proprietà `/allowAuthorized` deve essere impostata su `"0"`.

### Specifica dei documenti da memorizzare nella cache {#specifying-the-documents-to-cache}

La proprietà `/rules` controlla i documenti memorizzati nella cache in base al percorso del documento. Indipendentemente dalla proprietà `/rules`, Dispatcher non memorizza mai in cache un documento nelle seguenti circostanze:

* Se l’URI della richiesta contiene un punto interrogativo (`?`).
   * In genere indica una pagina dinamica, ad esempio un risultato di ricerca che non deve essere memorizzato nella cache.
* Se manca l’estensione del file.
   * Il server web ha bisogno dell’estensione per determinare il tipo di documento (tipo MIME).
* Se l’intestazione di autenticazione è impostata (configurabile)..
* Se l&#39;istanza AEM risponde con le seguenti intestazioni:

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>Dispatcher può memorizzare in cache i metodi GET o HEAD (per l’intestazione HTTP). Per ulteriori informazioni sul caching delle intestazioni di risposta, consulta la sezione [Memorizzazione in cache delle intestazioni di risposta HTTP](#caching-http-response-headers) .

Ogni elemento della proprietà `/rules` include un pattern [`glob`](#designing-patterns-for-glob-properties) e un tipo:

* Il pattern `glob` viene utilizzato per corrispondere al percorso del documento.
* Il tipo indica se memorizzare nella cache i documenti che corrispondono al pattern `glob`. Il valore può essere consentito (per memorizzare il documento nella cache) o negato (per eseguire sempre il rendering del documento).

Se non disponi di pagine dinamiche (oltre a quelle già escluse dalle regole di cui sopra), puoi configurare Dispatcher per memorizzare tutto nella cache. La sezione delle regole si presenta così:

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

Per informazioni sulle proprietà glob, vedere [Progettazione di modelli per proprietà glob](#designing-patterns-for-glob-properties).

Se alcune sezioni della pagina sono dinamiche (ad esempio un&#39;applicazione di notizie) o all&#39;interno di un gruppo di utenti chiuso, puoi definire le eccezioni:

>[!NOTE]
>
>I gruppi di utenti chiusi non devono essere memorizzati nella cache perché i diritti utente non vengono controllati per verificare la presenza di pagine memorizzate nella cache.

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**Compressione**

Nei server web Apache è possibile comprimere i documenti memorizzati nella cache. La compressione consente ad Apache di restituire il documento in formato compresso, se richiesto dal client. La compressione viene eseguita automaticamente abilitando il modulo Apache `mod_deflate`, ad esempio:

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

Utilizza la proprietà `/statfileslevel` per annullare la validità dei file memorizzati nella cache in base al loro percorso:

* Dispatcher crea `.stat`file in ogni cartella dalla cartella docroot al livello specificato. La cartella docroot è di livello 0.
* I file vengono invalidati toccando il file `.stat` . L&#39;ultima data di modifica del file `.stat` viene confrontata con l&#39;ultima data di modifica di un documento memorizzato nella cache. Il documento viene recuperato se il file `.stat` è più recente.

* Quando un file che si trova a un certo livello viene invalidato, dal docroot **a** vengono toccati il livello del file invalidato o il file `statsfilevel` configurato (a seconda di quale dei due valori sia inferiore).****`.stat`

   * Ad esempio: se imposti la proprietà `statfileslevel` su 6 e un file viene invalidato al livello 5, ogni file `.stat` da docroot a 5 verrà toccato. Continuando con questo esempio, se un file viene invalidato al livello 7 allora ogni . `stat` file da docroot a 6 sarà toccato (da allora  `/statfileslevel = "6"`).

Sono interessate solo le risorse **lungo il percorso** del file invalidato. Prendi in considerazione l’esempio seguente: un sito web utilizza la struttura `/content/myWebsite/xx/.` Se si imposta `statfileslevel` come 3, viene creato un file `.stat`come segue:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

Quando un file in `/content/myWebsite/xx` viene invalidato, viene toccato ogni file `.stat` dal punto di vista docroot a `/content/myWebsite/xx`. Questo vale solo per `/content/myWebsite/xx` e non per esempio `/content/myWebsite/yy` o `/content/anotherWebSite`.

>[!NOTE]
>
>L’annullamento della validità può essere impedito inviando un’ulteriore intestazione `CQ-Action-Scope:ResourceOnly`. Questo può essere utilizzato per svuotare determinate risorse senza invalidare altre parti della cache. Per ulteriori informazioni, consulta [questa pagina](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) e [Annullamento manuale della validità della cache del Dispatcher](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=en#configuring) .

>[!NOTE]
>
>Se si specifica un valore per la proprietà `/statfileslevel`, la proprietà `/statfile` viene ignorata.

### Annullamento automatico della validità dei file in cache {#automatically-invalidating-cached-files}

La proprietà `/invalidate` definisce i documenti che vengono invalidati automaticamente quando il contenuto viene aggiornato.

Con l’annullamento automatico della validità, Dispatcher non elimina i file memorizzati nella cache dopo un aggiornamento del contenuto, ma ne controlla la validità quando vengono successivamente richiesti. I documenti nella cache che non vengono invalidati automaticamente rimarranno nella cache fino a quando un aggiornamento del contenuto li eliminerà esplicitamente.

L’annullamento automatico della validità viene in genere utilizzato per le pagine HTML. Le pagine HTML spesso contengono collegamenti ad altre pagine, rendendo difficile determinare se un aggiornamento del contenuto influisce su una pagina. Per fare in modo che tutte le pagine rilevanti vengano invalidate quando il contenuto viene aggiornato, invalida automaticamente tutte le pagine HTML. La configurazione seguente invalida tutte le pagine HTML:

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

Per informazioni sulle proprietà glob, vedere [Progettazione di modelli per proprietà glob](#designing-patterns-for-glob-properties).

Questa configurazione causa la seguente attività quando `/content/wknd/us/en` è attivato:

* Tutti i file con pattern en.* vengono rimossi dalla cartella `/content/wknd/us`.
* La cartella `/content/wknd/us/en./_jcr_content` viene rimossa.
* Tutti gli altri file che corrispondono alla configurazione `/invalidate` non vengono eliminati immediatamente. Questi file vengono eliminati quando si verifica la richiesta successiva. Nel nostro esempio `/content/wknd.html` non viene eliminato, verrà eliminato quando viene richiesto `/content/wknd.html`.

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

La proprietà `/invalidateHandler` ti consente di definire uno script chiamato per ogni richiesta di invalidazione ricevuta da Dispatcher.

Viene chiamato con i seguenti argomenti:

* Handle : il percorso del contenuto invalidato
* Azione : l’azione di replica (ad esempio Attiva, Disattiva)
* Ambito azione - Ambito dell&#39;azione di replica (vuoto, a meno che non venga inviata un&#39;intestazione di `CQ-Action-Scope: ResourceOnly`, vedere [Annullamento della validità delle pagine in cache da AEM](page-invalidate.md) per ulteriori dettagli)

Può essere utilizzato per coprire diversi casi d’uso, ad esempio per annullare la validità di cache specifiche per altre applicazioni, o per gestire casi in cui l’URL esterno di una pagina e la sua posizione nel docroot non corrispondono al percorso del contenuto.

Di seguito è riportato uno script di esempio che registra ogni richiesta di annullamento della validità in un file.

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### script del gestore di invalidazione di esempio {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limitazione dei client in grado di svuotare la cache {#limiting-the-clients-that-can-flush-the-cache}

La proprietà `/allowedClients` definisce client specifici autorizzati a svuotare la cache. I modelli globbing sono confrontati con l&#39;IP.

Esempio:

1. nega l&#39;accesso a qualsiasi cliente
1. consente esplicitamente l&#39;accesso al localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

Per informazioni sulle proprietà glob, vedere [Progettazione di modelli per proprietà glob](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>È consigliabile definire il valore `/allowedClients`.
>
>In caso contrario, qualsiasi client può emettere una chiamata per cancellare la cache; se questa operazione viene eseguita ripetutamente, può avere un forte impatto sulle prestazioni del sito.

### Ignorare i parametri URL {#ignoring-url-parameters}

La sezione `ignoreUrlParams` definisce quali parametri URL vengono ignorati quando si determina se una pagina è memorizzata nella cache o distribuita dalla cache:

* Quando un URL di richiesta contiene parametri tutti ignorati, la pagina viene memorizzata nella cache.
* Quando un URL di richiesta contiene uno o più parametri che non vengono ignorati, la pagina non viene memorizzata nella cache.

Quando un parametro viene ignorato per una pagina, questa viene memorizzata nella cache la prima volta che la pagina viene richiesta. Le richieste successive per la pagina vengono servite nella cache, indipendentemente dal valore del parametro nella richiesta.

Per specificare quali parametri vengono ignorati, aggiungi regole glob alla proprietà `ignoreUrlParams` :

* Per ignorare un parametro, crea una proprietà glob che consenta il parametro.
* Per evitare che la pagina venga memorizzata nella cache, crea una proprietà glob che nega il parametro .

L’esempio seguente fa sì che Dispatcher ignori il parametro `q` , in modo che gli URL di richiesta che includono il parametro q siano memorizzati nella cache:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

Utilizzando il valore di esempio `ignoreUrlParams`, la seguente richiesta HTTP causa la memorizzazione nella cache della pagina perché il parametro `q` viene ignorato:

```xml
GET /mypage.html?q=5
```

Utilizzando il valore di esempio `ignoreUrlParams`, la seguente richiesta HTTP fa sì che la pagina sia **non** memorizzata nella cache perché il parametro `p` non viene ignorato:

```xml
GET /mypage.html?q=5&p=4
```

Per informazioni sulle proprietà glob, vedere [Progettazione di modelli per proprietà glob](#designing-patterns-for-glob-properties).

### Memorizzazione in cache delle intestazioni di risposta HTTP {#caching-http-response-headers}

>[!NOTE]
>
>Questa funzione è disponibile con la versione **4.1.11** di Dispatcher.

La proprietà `/headers` ti consente di definire i tipi di intestazione HTTP che verranno memorizzati nella cache da Dispatcher. Nella prima richiesta a una risorsa non memorizzata nella cache, tutte le intestazioni che corrispondono a uno dei valori configurati (vedi l&#39;esempio di configurazione seguente) vengono memorizzate in un file separato, accanto al file di cache. Nelle richieste successive alla risorsa memorizzata nella cache, le intestazioni memorizzate vengono aggiunte alla risposta.

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
>Inoltre, tieni presente che i caratteri globbing del file non sono consentiti. Per ulteriori dettagli, vedere [Progettazione di modelli per proprietà glob](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>Se hai bisogno che Dispatcher memorizzi e distribuisca le intestazioni di risposta ETag da AEM, procedi come segue:
>
>* Aggiungi il nome dell’intestazione nella sezione `/cache/headers`.
>* Aggiungi la seguente [direttiva Apache](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) nella sezione relativa a Dispatcher:

>
>
```xml
>FileETag none
>```

### Autorizzazioni per i file della cache del dispatcher {#dispatcher-cache-file-permissions}

La proprietà `mode` specifica quali autorizzazioni di file vengono applicate alle nuove directory e ai nuovi file nella cache. Questa impostazione è limitata dalla `umask` del processo chiamante. Si tratta di un numero ottale costruito dalla somma di uno o più dei seguenti valori:

* `0400` Consenti lettura per proprietario.
* `0200` Consenti scrittura per proprietario.
* `0100` Consenti al proprietario di eseguire ricerche nelle directory.
* `0040` Consenti lettura per membri del gruppo.
* `0020` Consenti scrittura per membri del gruppo.
* `0010` Consenti ai membri del gruppo di eseguire ricerche nella directory.
* `0004` Permetti agli altri di leggere.
* `0002` Permetti di scrivere da altri.
* `0001` Consente agli altri utenti di eseguire ricerche nella directory.

Il valore predefinito è `0755` che consente al proprietario di leggere, scrivere o cercare e al gruppo e ad altri di leggere o cercare.

### Limitazione del tocco di file .stat {#throttling-stat-file-touching}

Con la proprietà `/invalidate` predefinita, ogni attivazione invalida efficacemente tutti i file `.html` (quando il loro percorso corrisponde alla sezione `/invalidate`). Su un sito web con traffico considerevole, più attivazioni successive incrementeranno il carico di cpu sul backend. In questo caso, sarebbe auspicabile &quot;limitare&quot; il contatto di file `.stat` per mantenere reattivo il sito web. Per farlo, utilizza la proprietà `/gracePeriod` .

La proprietà `/gracePeriod` definisce il numero di secondi in cui una risorsa obsoleta e auto-invalidata può ancora essere servita dalla cache dopo l&#39;ultima attivazione in esecuzione. La proprietà può essere utilizzata in una configurazione in cui un batch di attivazioni in caso contrario annulla ripetutamente l&#39;intera cache. Il valore consigliato è 2 secondi.

Per ulteriori informazioni, consulta anche le sezioni `/invalidate` e `/statfileslevel`precedenti.

### Configurazione dell’annullamento della validità della cache basata sul tempo - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

Se impostata, la proprietà `/enableTTL` valuterà le intestazioni di risposta dal backend e, se contengono una `Cache-Control` età massima o `Expires` data, viene creato un file ausiliario vuoto accanto al file di cache, con un tempo di modifica uguale alla data di scadenza. Quando il file memorizzato nella cache viene richiesto oltre il tempo di modifica, viene automaticamente richiesto nuovamente dal backend.

>[!NOTE]
>
>Questa funzione è disponibile nella versione **4.1.11** o successiva del Dispatcher.

## Configurazione del bilanciamento del carico - /statistics {#configuring-load-balancing-statistics}

La sezione `/statistics` definisce categorie di file per i quali Dispatcher valuta la reattività di ciascun rendering. Dispatcher utilizza i punteggi per determinare quale render inviare una richiesta.

Ogni categoria creata definisce un pattern glob. Dispatcher confronta l’URI del contenuto richiesto con questi pattern per determinare la categoria del contenuto richiesto:

* L’ordine delle categorie determina l’ordine in cui vengono confrontate con l’URI.
* Il primo pattern di categoria che corrisponde all’URI è la categoria del file. Non vengono valutati altri pattern di categoria.

Dispatcher supporta un massimo di 8 categorie di statistiche. Se definisci più di 8 categorie, vengono utilizzate solo le prime 8.

**Selezione rendering**

Ogni volta che Dispatcher richiede una pagina di cui è stato eseguito il rendering, utilizza il seguente algoritmo per selezionare il rendering:

1. Se la richiesta contiene il nome di rendering in un cookie `renderid`, Dispatcher lo utilizza.
1. Se la richiesta non include un cookie `renderid`, Dispatcher confronta le statistiche di rendering:

   1. Dispatcher determina la categoria dell’URI della richiesta.
   1. Dispatcher determina quale render ha il punteggio di risposta più basso per quella categoria e lo seleziona.

1. Se non è ancora stato selezionato alcun rendering, utilizza il primo rendering nell’elenco.

Il punteggio per la categoria di un rendering si basa sui tempi di risposta precedenti, nonché sulle connessioni non riuscite e riuscite precedenti tentate da Dispatcher. Per ogni tentativo, viene aggiornato il punteggio per la categoria dell’URI richiesto.

>[!NOTE]
>
>Se non si utilizza il bilanciamento del carico, è possibile omettere questa sezione.

### Definizione delle categorie di statistiche {#defining-statistics-categories}

Definire una categoria per ciascun tipo di documento per il quale si desidera conservare le statistiche per la selezione del rendering. La sezione `/statistics` contiene una sezione `/categories`. Per definire una categoria, aggiungi una riga sotto la sezione `/categories` con il seguente formato:

`/name { /glob "pattern"}`

La categoria `name` deve essere univoca per la farm. La sezione `pattern` è descritta nella sezione [Progettazione di modelli per proprietà glob](#designing-patterns-for-glob-properties) .

Per determinare la categoria di un URI, Dispatcher confronta l’URI con ciascun pattern di categoria fino a quando non viene trovata una corrispondenza. Dispatcher inizia con la prima categoria nell’elenco e continua in ordine. Quindi, inserire prima le categorie con modelli più specifici.

Ad esempio, Dispatcher il file `dispatcher.any` predefinito definisce una categoria HTML e un’altra categoria. La categoria HTML è più specifica e quindi viene visualizzata per prima:

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

### Riflesso dell’indisponibilità del server nelle statistiche del Dispatcher {#reflecting-server-unavailability-in-dispatcher-statistics}

La proprietà `/unavailablePenalty` imposta il tempo (in decimi di secondo) applicato alle statistiche di rendering quando una connessione al rendering non riesce. Dispatcher aggiunge l’ora alla categoria di statistiche che corrisponde all’URI richiesto.

Ad esempio, la sanzione viene applicata quando non è possibile stabilire la connessione TCP/IP al nome host/porta designato, perché AEM non è in esecuzione (e non in ascolto) o a causa di un problema relativo alla rete.

La proprietà `/unavailablePenalty` è un elemento figlio diretto della sezione `/farm` (un elemento di pari livello della sezione `/statistics`).

Se non esiste alcuna proprietà `/unavailablePenalty`, viene utilizzato un valore di `"1"`.

```xml
/unavailablePenalty "1"
```

## Identificazione di una cartella di connessione permanenti - /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

La proprietà `/stickyConnectionsFor` definisce una cartella contenente documenti permanenti; questo sarà accessibile tramite l’URL . Dispatcher invia tutte le richieste, da un singolo utente, presenti in questa cartella alla stessa istanza di rendering. Le connessioni permanenti garantiscono la presenza e la coerenza dei dati della sessione per tutti i documenti. Questo meccanismo utilizza il cookie `renderid` .

L’esempio seguente definisce una connessione fissa alla cartella /products :

```xml
/stickyConnectionsFor "/products"
```

Quando una pagina è composta da contenuto proveniente da più nodi di contenuto, includi la proprietà `/paths` che elenca i percorsi del contenuto. Ad esempio, una pagina contiene il contenuto di `/content/image`, `/content/video` e `/var/files/pdfs`. La seguente configurazione abilita connessioni permanenti per tutti i contenuti della pagina:

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

Quando le connessioni permanenti sono abilitate, il modulo dispatcher imposta il cookie `renderid`. Questo cookie non ha il flag `httponly` che deve essere aggiunto per migliorare la sicurezza. Per farlo, imposta la proprietà `httpOnly` nel nodo `/stickyConnections` di un file di configurazione `dispatcher.any`. Il valore della proprietà (sia `0` che `1`) definisce se al cookie `renderid` è stato aggiunto l&#39;attributo `HttpOnly`. Il valore predefinito è `0`, il che significa che l’attributo non verrà aggiunto.

Per ulteriori informazioni sul flag `httponly`, leggere [questa pagina](https://www.owasp.org/index.php/HttpOnly).

### sicuro {#secure}

Quando le connessioni permanenti sono abilitate, il modulo dispatcher imposta il cookie `renderid`. Questo cookie non ha il flag `secure` che deve essere aggiunto per migliorare la sicurezza. Per farlo, imposta la proprietà `secure` nel nodo `/stickyConnections` di un file di configurazione `dispatcher.any`. Il valore della proprietà (sia `0` che `1`) definisce se al cookie `renderid` è stato aggiunto l&#39;attributo `secure`. Il valore predefinito è `0`, il che significa che l’attributo verrà aggiunto **se** la richiesta in arrivo è sicura. Se il valore è impostato su `1`, il flag Secure verrà aggiunto indipendentemente dal fatto che la richiesta in arrivo sia protetta o meno.

## Gestione degli errori di connessione del rendering {#handling-render-connection-errors}

Configura il comportamento del Dispatcher quando il server di rendering restituisce un errore 500 o non è disponibile.

### Specifica di una pagina di verifica dello stato {#specifying-a-health-check-page}

Utilizzare la proprietà `/health_check` per specificare un URL controllato quando si verifica un codice di stato 500. Se questa pagina restituisce anche un codice di stato 500, l’istanza viene considerata non disponibile e al rendering viene applicata una penalità di tempo configurabile ( `/unavailablePenalty`) prima di riprovare.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifica del ritardo del tentativo di pagina {#specifying-the-page-retry-delay}

La proprietà `/retryDelay` imposta il tempo (in secondi) in cui Dispatcher attende tra i turni di tentativi di connessione con i render della farm. Per ogni turno, il numero massimo di tentativi di connessione a un rendering da parte di Dispatcher è il numero di rendering nella farm.

Dispatcher utilizza un valore di `"1"` se `/retryDelay` non è definito in modo esplicito. Il valore predefinito è appropriato nella maggior parte dei casi.

```xml
/retryDelay "1"
```

### Configurazione del numero di tentativi {#configuring-the-number-of-retries}

La proprietà `/numberOfRetries` imposta il numero massimo di turni di tentativi di connessione eseguiti da Dispatcher con i render. Se Dispatcher non riesce a connettersi a un rendering dopo questo numero di tentativi, Dispatcher restituisce una risposta non riuscita.

Per ogni turno, il numero massimo di tentativi di connessione a un rendering da parte di Dispatcher è il numero di rendering nella farm. Pertanto, il numero massimo di tentativi di connessione da parte di Dispatcher è ( `/numberOfRetries`) x (il numero di rendering).

Se il valore non è definito in modo esplicito, il valore predefinito è `5`.

```xml
/numberOfRetries "5"
```

### Utilizzo del meccanismo di failover {#using-the-failover-mechanism}

Abilita il meccanismo di failover nella farm di Dispatcher per inviare nuovamente le richieste a render diversi quando la richiesta originale non riesce. Quando il failover è abilitato, Dispatcher ha il seguente comportamento:

* Quando una richiesta a un rendering restituisce lo stato HTTP 503 (NON DISPONIBILE), Dispatcher invia la richiesta a un rendering diverso.
* Quando una richiesta a un rendering restituisce lo stato HTTP 50x (diverso da 503), Dispatcher invia una richiesta per la pagina configurata per la proprietà `health_check` .
   * Se il controllo di integrità restituisce 500 (INTERNAL_SERVER_ERROR), Dispatcher invia la richiesta originale a un rendering diverso.
   * Se il controllo di integrità restituisce lo stato HTTP 200, Dispatcher restituisce l’errore HTTP 500 iniziale al client.

Per abilitare il failover, aggiungi la seguente riga alla farm (o al sito Web):

```xml
/failover "1"
```

>[!NOTE]
>
>Per riprovare le richieste HTTP che contengono un corpo, Dispatcher invia un’intestazione di richiesta `Expect: 100-continue` al rendering prima di eseguire lo spooling del contenuto effettivo. CQ 5.5 con CQSE risponde immediatamente con 100 (CONTINUE) o con un codice di errore. Anche altri contenitori servlet dovrebbero supportare questo.

## Ignorare gli errori di interruzione - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>Questa opzione di solito non è necessaria. Devi utilizzarlo solo quando vedi i seguenti messaggi di log:
>
>`Error while reading response: Interrupted system call`

Qualsiasi chiamata di sistema orientata al file system può essere interrotta `EINTR` se l&#39;oggetto della chiamata di sistema si trova su un sistema remoto accessibile tramite NFS. Se queste chiamate di sistema possono essere interrotte o interrotte, si basa sul modo in cui il file system sottostante è stato montato sul computer locale.

Usa il parametro `/ignoreEINTR` se la tua istanza dispone di tale configurazione e il registro contiene il seguente messaggio:

`Error while reading response: Interrupted system call`

Internamente, Dispatcher legge la risposta dal server remoto (cioè AEM) utilizzando un loop che può essere rappresentato come:

```text
while (response not finished) {  
read more data  
}
```

Tali messaggi possono essere generati quando `EINTR` si trova nella sezione &quot; `read more data`&quot; e sono causati dalla ricezione di un segnale prima della ricezione di qualsiasi dato.

Per ignorare tali interruzioni è possibile aggiungere il seguente parametro a `dispatcher.any` (prima di `/farms`):

`/ignoreEINTR "1"`

Se si imposta `/ignoreEINTR` su `"1"`, Dispatcher continua a tentare di leggere i dati fino a quando non viene letta la risposta completa. Il valore predefinito è `0` e disattiva l’opzione.

## Progettazione di modelli per proprietà glob {#designing-patterns-for-glob-properties}

Diverse sezioni nel file di configurazione di Dispatcher utilizzano le proprietà `glob` come criteri di selezione per le richieste client. I valori delle proprietà `glob` sono pattern confrontati con un aspetto della richiesta, ad esempio il percorso della risorsa richiesta o l’indirizzo IP del client. Ad esempio, gli elementi nella sezione `/filter` utilizzano i pattern `glob` per identificare i percorsi delle pagine su cui Dispatcher agisce o rifiuta.

I valori `glob` possono includere caratteri jolly e caratteri alfanumerici per definire il pattern.

| Carattere jolly | Descrizione | Esempi |
|--- |--- |--- |
| `*` | Corrisponde a zero o più istanze contigue di qualsiasi carattere nella stringa. Il carattere finale della corrispondenza è determinato da una delle situazioni seguenti: <br/>Un carattere nella stringa corrisponde al carattere successivo nel pattern e il carattere del pattern ha le seguenti caratteristiche:<br/><ul><li>Non è un *</li><li>Non è un ?</li><li>Un carattere letterale (incluso uno spazio) o una classe di caratteri.</li><li>Viene raggiunta la fine del pattern.</li></ul>All&#39;interno di una classe di caratteri, il carattere viene interpretato letteralmente. | `*/geo*` Corrisponde a qualsiasi pagina sotto il  `/content/geometrixx` nodo e il  `/content/geometrixx-outdoors` nodo. Le seguenti richieste HTTP corrispondono al pattern glob: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>Corrisponde a qualsiasi pagina sotto il  `/content/geometrixx-outdoors` nodo. Ad esempio, la seguente richiesta HTTP corrisponde al pattern glob: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | Corrisponde a qualsiasi carattere singolo. Utilizzare classi di caratteri esterne. All&#39;interno di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*outdoors/??/*`<br/> Corrisponde alle pagine di qualsiasi lingua del sito geometrixx-outdoors. Ad esempio, la seguente richiesta HTTP corrisponde al pattern glob: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>La seguente richiesta non corrisponde al pattern glob:  <br/><ul><li>&quot;GET /content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | Richiama l&#39;inizio e la fine di una classe di caratteri. Le classi di caratteri possono includere uno o più intervalli di caratteri e caratteri singoli.<br/>Una corrispondenza si verifica se il carattere di destinazione corrisponde a uno qualsiasi dei caratteri della classe di caratteri o all&#39;interno di un intervallo definito.<br/>Se la staffa di chiusura non è inclusa, il pattern non produce alcuna corrispondenza. | `*[o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>Corrisponde alle seguenti richieste HTTP:  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | Indica un intervallo di caratteri. Da utilizzare nelle classi di caratteri.  Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[m-p]men.html*` Corrisponde alla seguente richiesta HTTP:  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | Ignora la classe di carattere o di carattere che segue. Utilizzare solo per negare caratteri e intervalli di caratteri all&#39;interno delle classi di caratteri. Equivalente a `^ wildcard`. <br/>Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | `*[!o]men.html*`<br/> Corrisponde alla seguente richiesta HTTP:  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>Non corrisponde alla seguente richiesta HTTP:  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> Non corrisponde alla seguente richiesta HTTP:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` o `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | Ignora il carattere o l&#39;intervallo di caratteri che segue. Da utilizzare per negare solo caratteri e intervalli di caratteri all’interno delle classi di caratteri. Equivalente al carattere jolly `!`. <br/>Al di fuori di una classe di caratteri, questo carattere viene interpretato letteralmente. | Gli esempi per il carattere jolly `!` vengono applicati, sostituendo i caratteri `!` nei pattern di esempio con caratteri `^`. |


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

* Posizione del file di registro del Dispatcher.
* Livello di log.

Per ulteriori informazioni, consulta la documentazione del server web e il file readme dell’istanza di Dispatcher .

**Registri ruotati/tubati Apache**

Se utilizzi un server web **Apache** puoi utilizzare la funzionalità standard per i registri ruotati e/o con tubazioni. Ad esempio, utilizzando i registri piping:

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

Questo verrà ruotato automaticamente:

* il file di registro del dispatcher; con una marca temporale nell’estensione (`logs/dispatcher.log%Y%m%d`).
* su base settimanale (60 x 60 x 24 x 7 = 604800 secondi).

Vedi la documentazione del server web Apache su Log Rotation e Piped Logs; ad esempio [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>Al momento dell’installazione, il livello di registro predefinito è alto (ovvero livello 3 = Debug), in modo che Dispatcher registri tutti gli errori e gli avvisi. Ciò è molto utile nelle fasi iniziali.
>
>Tuttavia, questo richiede risorse aggiuntive, quindi quando Dispatcher funziona senza problemi *in base alle tue esigenze*, puoi (dovrebbe) abbassare il livello di registro.

### Registrazione traccia {#trace-logging}

Tra gli altri miglioramenti apportati al Dispatcher, la versione 4.2.0 introduce anche la funzione di registrazione della traccia.

Si tratta di un livello più alto della registrazione di debug, che mostra informazioni aggiuntive nei registri. Aggiunge la registrazione per:

* i valori delle intestazioni inoltrate;
* La regola applicata per una determinata azione.

Puoi abilitare la funzione Trace Logging impostando il livello di registro su `4` nel server web.

Di seguito è riportato un esempio di log con tracciamento abilitato:

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

Per confermare il funzionamento e l’interazione di base del server web, del Dispatcher e dell’istanza di AEM, procedi come segue:

1. Imposta `loglevel` su `3`.

1. Avviare il server web; viene avviato anche Dispatcher.
1. Avvia l&#39;istanza AEM.
1. Controlla i file di registro e di errore per il tuo server web e il Dispatcher.
   * A seconda del server web, dovresti visualizzare messaggi quali:
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` e
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. Naviga sul sito web tramite il server web. Conferma che il contenuto venga visualizzato come necessario.\
   Ad esempio, in un’installazione locale in cui AEM eseguito sulla porta `4502` e sul server Web in `80` accedi alla console Siti web utilizzando entrambi:
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * I risultati dovrebbero essere identici. Conferma l’accesso ad altre pagine con lo stesso meccanismo.

1. Verifica che la directory della cache sia stata riempita.
1. Attiva una pagina per verificare che la cache venga scaricata correttamente.
1. Se tutto funziona correttamente, puoi ridurre il `loglevel` a `0`.

## Utilizzo di più istanze di Dispatcher {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più istanze di Dispatcher. Ad esempio, puoi utilizzare:

* un’istanza di Dispatcher per pubblicare un sito web nella Intranet;
* una seconda istanza di Dispatcher, con un indirizzo diverso e impostazioni di sicurezza diverse, per pubblicare lo stesso contenuto in Internet.

In tal caso, assicurati che ogni richiesta venga gestita tramite un’unica istanza di Dispatcher. Un’istanza di Dispatcher non gestisce le richieste provenienti da un’altra istanza di Dispatcher. Assicurati pertanto che entrambe le istanze di Dispatcher accedano direttamente al sito web AEM.

## Debug {#debugging}

Quando aggiungi l’intestazione `X-Dispatcher-Info` a una richiesta, Dispatcher risponde se la destinazione è stata memorizzata nella cache, restituita dalla cache o meno. L&#39;intestazione della risposta `X-Cache-Info` contiene queste informazioni in un formato leggibile. Puoi utilizzare queste intestazioni di risposta per eseguire il debug dei problemi relativi alle risposte memorizzate nella cache di Dispatcher.

Questa funzionalità non è abilitata per impostazione predefinita, pertanto affinché l’intestazione di risposta `X-Cache-Info` sia inclusa, la farm deve contenere la seguente voce:

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

Inoltre, l&#39;intestazione `X-Dispatcher-Info` non ha bisogno di un valore, ma se utilizzi `curl` per il test devi fornire un valore per inviare l&#39;intestazione, ad esempio:

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

Di seguito è riportato un elenco contenente le intestazioni di risposta che `X-Dispatcher-Info` restituiranno:

* **memorizzato nella cache**\
   Il file di destinazione è contenuto nella cache e il dispatcher ha stabilito che è valido per consegnarlo.
* **caching**\
   Il file di destinazione non è contenuto nella cache e il dispatcher ha stabilito che è valido memorizzare nella cache l&#39;output e consegnarlo.
* **memorizzazione in cache: Il file stat è più**
recenteIl file target è contenuto nella cache, tuttavia viene invalidato da un file stat più recente. Il dispatcher eliminerà il file di destinazione, lo ricreerà dall’output e lo distribuirà.
* **non memorizzabile in cache: nessun document**
rootLa configurazione della farm non contiene un document root (elemento di configurazione 
`cache.docroot`).
* **non memorizzabile in cache: percorso del file di cache troppo lungo**\
   Il file di destinazione, ovvero la concatenazione della directory principale del documento e del file URL, supera il nome file più lungo possibile sul sistema.
* **non memorizzabile in cache: percorso file temporaneo troppo lungo**\
   Il modello di nome file temporaneo supera il nome file più lungo possibile sul sistema. Il dispatcher crea prima un file temporaneo prima di creare o sovrascrivere effettivamente il file memorizzato nella cache. Il nome del file temporaneo è il nome del file di destinazione con i caratteri `_YYYYXXXXXX` aggiunti alla fine, in cui verranno sostituiti i caratteri `Y` e `X` per creare un nome univoco.
* **non memorizzabile in cache: l&#39;URL della richiesta non ha estensione**\
   L&#39;URL della richiesta non ha un&#39;estensione oppure esiste un percorso che segue l&#39;estensione del file, ad esempio: `/test.html/a/path`.
* **non memorizzabile in cache: La richiesta non era un metodo GET o**
HEADTil metodo HTTP non è né un GET né un HEAD. Il dispatcher presuppone che l’output conterrà dati dinamici che non devono essere memorizzati nella cache.
* **non memorizzabile in cache: la richiesta conteneva una stringa di interrogazione**\
   La richiesta conteneva una stringa di interrogazione. Il dispatcher presuppone che l’output dipenda dalla stringa di query specificata e quindi non memorizza in cache.
* **non memorizzabile in cache: session manager non autenticato**\
   La cache della farm è gestita da un gestore di sessione (la configurazione contiene un nodo `sessionmanagement`) e la richiesta non conteneva le informazioni di autenticazione appropriate.
* **non memorizzabile in cache: richiesta di autorizzazione**\
   La farm non può memorizzare nella cache l&#39;output ( `allowAuthorized 0`) e la richiesta contiene informazioni di autenticazione.
* **non memorizzabile in cache: target è una directory**\
   Il file di destinazione è una directory. Questo potrebbe indicare un errore concettuale, in cui un URL e alcuni URL secondari contengono entrambi un output memorizzabile nella cache, ad esempio se una richiesta a `/test.html/a/file.ext` viene prima e contiene un output memorizzabile nella cache, il dispatcher non sarà in grado di memorizzare l’output di una richiesta successiva a `/test.html`.
* **non memorizzabile in cache: l&#39;URL della richiesta ha una barra finale**\
   L’URL della richiesta ha una barra finale.
* **non memorizzabile in cache: URL della richiesta non presente nelle regole della cache**\
   Le regole di cache della farm negano esplicitamente la memorizzazione in cache dell’output di alcuni URL di richiesta.
* **non memorizzabile in cache: accesso negato tramite controllo autorizzazioni**\
   Il controllo autorizzazioni della farm ha negato l&#39;accesso al file memorizzato nella cache.
* **non memorizzabile in cache: sessione non**
validaLa cache della farm è gestita da un gestore di sessione (la configurazione contiene un  `sessionmanagement` nodo) e la sessione dell&#39;utente non è o non è più valida.
* **non memorizzabile in cache: La risposta contiene`no_cache`**
il server remoto ha restituito un 
`Dispatcher: no_cache` intestazione, divieto del dispatcher di memorizzare nella cache l&#39;output.
* **non memorizzabile in cache: la lunghezza del contenuto della risposta è**
zeroLa lunghezza del contenuto della risposta è zero; il dispatcher non creerà un file di lunghezza zero.
