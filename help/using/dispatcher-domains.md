---
title: 'Utilizzo di Dispatcher con più domini '
seo-title: 'Utilizzo di Dispatcher con più domini '
description: Scopri come utilizzare Dispatcher per elaborare le richieste di pagina in più domini Web.
seo-description: Scopri come utilizzare Dispatcher per elaborare le richieste di pagina in più domini Web.
uuid: 7342 a 1 c 2-fe 61-49 be-a 240-b 487 d 53 c 7 ec 1
contentOwner: Utente
cq-exporttemplate: /etc/contentsync/templates/geometrixx/page/rewrite
products: SG_ EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: riferimento
discoiquuid: 40 d 91 d 66-c 99 b -422 d -8 e 61-c 0 ced 23272 ef
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# Utilizzo di Dispatcher con più domini {#using-dispatcher-with-multiple-domains}

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Se hai seguito un collegamento alla documentazione di Dispatcher incorporata nella documentazione di AEM o CQ, potresti essere stato reindirizzato a questa pagina.

Usa il dispatcher per elaborare le richieste di pagina in più domini Web, fornendo al contempo le condizioni seguenti:

* Il contenuto Web per entrambi i domini è memorizzato in un unico archivio AEM.
* I file nella cache del dispatcher possono essere invalidati separatamente per ogni dominio.

Ad esempio, un&#39;azienda pubblica siti Web per due marchi: Brand A e Brand B. Il contenuto delle pagine del sito Web viene creato in AEM e memorizzato nella stessa area di lavoro archivio:

```
/
| - content  
   | - sitea  
       | - content nodes  
   | - siteb  
       | - content nodes
```

Le pagine per `BrandA.com` sono memorizzate sotto `/content/sitea`. Le richieste client per l&#39;URL `https://BrandA.com/en.html` restituiscono la pagina sottoposta a rendering per il `/content/sitea/en` nodo. Analogamente, le pagine per sono `BrandB.com` memorizzate di seguito `/content/siteb`.

Quando si utilizza Dispatcher per memorizzare il contenuto nella cache, le associazioni devono essere effettuate tra l&#39;URL della pagina nella richiesta HTTP client, il percorso del file corrispondente nella cache e il percorso del file corrispondente nella directory archivio.

## Richieste client

Quando i client inviano richieste HTTP al server Web, l&#39;URL della pagina richiesta deve essere risolto al contenuto nella cache del dispatcher e quindi al contenuto della directory archivio.

![](assets/chlimage_1-8.png)

1. Il sistema del nome di dominio rileva l&#39;indirizzo IP del server Web registrato per il nome di dominio nella richiesta HTTP.
1. La richiesta HTTP viene inviata al server Web.
1. La richiesta HTTP viene passata al dispatcher.
1. Il dispatcher determina se i file memorizzati nella cache sono validi. Se valido, i file memorizzati nella cache vengono serviti al client.
1. Se i file memorizzati nella cache non sono validi, il dispatcher richiede le pagine di recente rendering dall&#39;istanza di pubblicazione AEM.

## Annullamento validità cache

Quando agenti di replica Svuotamento del dispatcher richiedono che il dispatcher invalidi i file memorizzati nella cache, il percorso del contenuto nell&#39;archivio deve risolvere il contenuto della cache.

![](assets/chlimage_1-9.png)

1. Una pagina viene attivata nell&#39;istanza di creazione AEM e il contenuto viene replicato nell&#39;istanza di pubblicazione.
1. Il Dispatcher Svuota il dispatcher chiama il dispatcher per annullare la validità della cache per il contenuto replicato.
1. Il dispatcher tocca uno o più file. stat per annullare la validità dei file memorizzati nella cache.

Per utilizzare Dispatcher con più domini, devi configurare AEM, Dispatcher e il server Web. Le soluzioni descritte in questa pagina sono generali e si applicano alla maggior parte degli ambienti. A causa della complessità di alcune topologie AEM, la soluzione può richiedere ulteriori configurazioni personalizzate per risolvere problemi specifici. Probabilmente dovrete adattare gli esempi per soddisfare i criteri di infrastruttura e gestione IT esistenti.

## Mappatura URL {#url-mapping}

Per abilitare gli URL di dominio e i percorsi di contenuto per risolvere i file memorizzati nella cache, in un certo momento nel processo un percorso file o un URL di pagina deve essere convertito. Sono fornite descrizioni delle seguenti strategie comuni, in cui le traduzioni di percorso o URL avvengono in diversi punti del processo:

* (Consigliato) L&#39;istanza di pubblicazione AEM utilizza la mappatura Sling per la risoluzione delle risorse per implementare regole di riscrittura URL interne. Gli URL di dominio vengono convertiti in percorsi repository di contenuto. (consultate [Riscrivere gli URL in entrata](dispatcher-domains.md#main-pars-title-2).)
* Il server Web utilizza regole interne di riscrittura URL che traducono gli URL dominio in percorsi cache. (Il [server Web riscrive gli URL in entrata](dispatcher-domains.md#main-pars-title-1)).

In genere è consigliabile utilizzare URL brevi per le pagine Web. In genere, gli URL delle pagine riflettono la struttura delle cartelle archivio contenenti il contenuto Web. Tuttavia, gli URL non mostrano i nodi directory principale, ad esempio `/content`. Il client non è necessariamente a conoscenza della struttura dell&#39;archivio AEM.

## Requisiti generali {#general-requirements}

L&#39;ambiente deve implementare le configurazioni seguenti per supportare il dispatcher che lavora con più domini:

* Il contenuto di ciascun dominio risiede in rami separati della directory archivio (vedete l&#39;ambiente di esempio di seguito).
* L&#39;agente replica di Svuotamento del dispatcher è configurato nell&#39;istanza di pubblicazione AEM. (Vedere [Annullamento della cache del dispatcher da un&#39;istanza di pubblicazione](page-invalidate.md).)
* Il sistema nome dominio risolve i nomi di dominio all&#39;indirizzo IP del server Web.
* La cache del dispatcher riflette la struttura di directory dell&#39;archivio di contenuto AEM. I percorsi dei file sotto la directory principale del documento del server Web sono identici ai percorsi dei file nella directory archivio.

## Ambiente per gli esempi forniti {#environment-for-the-provided-examples}

Le soluzioni di esempio fornite sono valide per un ambiente con le seguenti caratteristiche:

* Le istanze di creazione e pubblicazione di AEM vengono distribuite sui sistemi Linux.
* Apache HTTPD è il server Web, implementato su un sistema Linux.
* L&#39;archivio di contenuto AEM e la directory principale del documento del server Web utilizzano le seguenti strutture dei file (la directory principale del documento Apache è/`usr/lib/apache/httpd-2.4.3/htdocs)`:

   **Archivio**

```
  | - /content  
    | - sitea  
  |    | - content nodes 
    | - siteb  
       | - conent nodes
```

**Cartella principale documento del server Web**

```
  | - /usr  
    | - lib  
      | - apache  
        | - httpd-2.4.3  
          | - htdocs  
            | - content  
              | - sitea  
                 | - content nodes 
              | - siteb  
                 | - content nodes
```

## AEM riscrive gli URL in arrivo {#aem-rewrites-incoming-urls}

La mappatura sling per la risoluzione delle risorse consente di associare URL in arrivo ai percorsi di contenuto AEM. Crea mappature sull&#39;istanza di pubblicazione AEM in modo che le richieste di rendering del dispatcher vengano risolte nel contenuto corretto della directory archivio.

Le richieste di dispatcher per il rendering della pagina identificano la pagina utilizzando l&#39;URL passato dal server Web. Quando l&#39;URL include un nome di dominio, le mappature Sling risolvono l&#39;URL del contenuto. L&#39;elemento grafico seguente illustra una mappatura dell&#39; `branda.com/en.html` URL sul `/content/sitea/en` nodo.

![](assets/chlimage_1-10.png)

La cache del dispatcher riflette la struttura del nodo archivio. Pertanto, quando si verificano delle attivazioni di pagina, le richieste risultanti per inco la pagina nella cache non richiedono la traduzione di URL o percorsi.

![](assets/chlimage_1-11.png)

## Definire gli ospitanti virtuali sul server Web {#define-virtual-hosts-on-the-web-server}

Definite gli ospitanti virtuali sul server Web in modo che un&#39;altra cartella principale del documento possa essere assegnata a ogni dominio Web:

* Il server Web deve definire un dominio virtuale per ciascun dominio Web.
* Per ciascun dominio, configurate la radice del documento affinché coincida con la cartella nella directory archivio che contiene il contenuto Web del dominio.
* Ogni dominio virtuale deve includere anche configurazioni correlate al dispatcher, come descritto nella pagina [Installazione del dispatcher](dispatcher-install.md) .

Il seguente file di esempio `httpd.conf` configura due domini virtuali per un server Web Apache:

* I nomi dei server (che coincidono con i nomi di dominio) sono branda.com (riga 16) e brandb.com (riga 30).
* La directory principale del documento di ogni dominio virtuale è la directory nella cache del dispatcher che contiene le pagine del sito. (righe 17 e 31)

Con questa configurazione, quando viene riccata una richiesta per `https://branda.com/en/products.html`:

* Associa l&#39;URL all&#39;host virtuale con `ServerName` un `branda.com.`

* Inoltrare l&#39;URL al dispatcher.

### httpd. conf {#httpd-conf}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 0
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

Gli ospitanti virtuali ereditano il [valore](dispatcher-install.md#main-pars-67-table-7) della proprietà dispatcherconfig che è configurato nella sezione server principale. Gli host virtuali possono includere la propria proprietà dispatcherconfig per ignorare la configurazione del server principale.

### Configurare il dispatcher per gestire più domini {#configure-dispatcher-to-handle-multiple-domains}

Per supportare URL che includono nomi di dominio e i relativi host virtuali corrispondenti, definite le seguenti aziende di Dispatcher:

* Configurate una fattoria di Dispatcher per ciascun host virtuale. Queste richieste di processo vengono richieste dal server Web per ciascun dominio, controllate i file memorizzati nella cache e le pagine dai rendering.
* Configurate una fattoria di Dispatcher utilizzata per annullare la validità del contenuto della cache, indipendentemente dal dominio a cui appartiene il contenuto. Questa fattoria gestisce le richieste di annullamento della validità dei file dagli agenti di replica di Svuotamento del dispatcher.

### Creare allevamenti di dispatcher per gli ospitanti virtuali

Le aziende per gli host virtuali devono disporre delle configurazioni seguenti in modo che gli URL delle richieste HTTP client vengano risolti nei file corretti nella cache del dispatcher:

* La `/virtualhosts` proprietà è impostata sul nome del dominio. Questa proprietà consente a Dispatcher di associare la fattoria al dominio.
* La `/filter` proprietà consente l&#39;accesso al percorso dell&#39;URL di richiesta troncato dopo la parte del nome del dominio. Ad esempio, per l&#39; `https://branda.com/en.html` URL il percorso viene interpretato come `/en.html`, pertanto il filtro deve consentire l&#39;accesso a questo percorso.

* La `/docroot` proprietà viene impostata sul percorso della directory principale del contenuto del sito del dominio nella cache del dispatcher. Questo percorso viene utilizzato come prefisso per l&#39;URL concatenato dalla richiesta originale. Ad esempio, il dopot of `/usr/lib/apache/httpd-2.4.3/htdocs/sitea` cause la richiesta `https://branda.com/en.html` di risoluzione al `/usr/lib/apache/httpd-2.4.3/htdocs/sitea/en.html` file.

Inoltre, l&#39;istanza di pubblicazione AEM deve essere designata come rendering per l&#39;host virtuale. Configura le altre proprietà della fattoria, a seconda delle necessità. Il codice seguente è una configurazione farm abbreviata per il dominio branda.com:

```xml
/farm_sitea  {     
    ...
    /virtualhosts { "branda.com" }
    /renders {
      /rend01  { /hostname "127.0.0.1"  /port "4503" }
    }
    /filter {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/en*" }  
      ...
     }
    /cache {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs/content/sitea"
      ...
   }
   ...
}
```

### Creare una fattoria di dispatcher per la convalida della cache

Una fattoria di Dispatcher è necessaria per gestire le richieste di annullamento della validità dei file memorizzati nella cache. Questa fattoria deve essere in grado di accedere ai file. stat nelle directory docroot di ciascun host virtuale.

Le seguenti configurazioni di proprietà consentono al dispatcher di risolvere i file nell&#39;archivio di contenuto AEM dai file nella cache:

* La `/docroot` proprietà è impostata sul docroot predefinito del server Web. In genere, questa è la directory in cui viene creata `/content` la cartella. Un valore di esempio per Apache su Linux `/usr/lib/apache/httpd-2.4.3/htdocs`è.
* La `/filter` proprietà consente l&#39;accesso ai file sotto la `/content` directory.

La `/statfileslevel`proprietà deve essere sufficientemente alta, in modo che i file. stat vengano creati nella directory principale di ciascun host virtuale. Questa proprietà consente di invalidare separatamente la cache di ciascun dominio. Per l&#39;impostazione di esempio, un `/statfileslevel` valore di `2` crea file. stat nella `*docroot*/content/sitea` directory e nella `*docroot*/content/siteb` directory.

Inoltre, l&#39;istanza di pubblicazione deve essere designata come rendering per l&#39;host virtuale. Configura le altre proprietà della fattoria, a seconda delle necessità. Il codice seguente è una configurazione abbreviata per la fattoria utilizzata per annullare la validità della cache:

```xml
/farm_flush {  
    ...
    /virtualhosts   { "invalidation_only" }
    /renders  {
      /rend01  { /hostname "127.0.0.1" /port "4503" }
    }
    /filter   {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" } 
      ...
      }
    /cache  {
       /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
       /statfileslevel "2"
       ...
   }
   ...
}
```

Quando avviate il server Web, il registro del dispatcher (in modalità debug) indica l&#39;inizializzazione di tutte le aziende:

```shell
Dispatcher initializing (build 4.1.2)
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_sitea].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_siteb].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_flush].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs
[Fri Nov 02 16:27:18 2012] [I] [24974(140006182991616)] Dispatcher initialized (build 4.1.2)
```

### Configurare la mappatura Sling per la risoluzione delle risorse {#configure-sling-mapping-for-resource-resolution}

Utilizzate la mappatura Sling per la risoluzione delle risorse in modo che gli URL basati sul dominio risolvono il contenuto nell&#39;istanza di pubblicazione AEM. La mappatura delle risorse converte gli URL in arrivo dal dispatcher (originariamente dalle richieste HTTP client) ai nodi contenuto.

Per informazioni sulla mappatura delle risorse Sling, consultate [Mappature per Risoluzione](https://sling.apache.org/site/mappings-for-resource-resolution.html) delle risorse nella documentazione Sling.

In genere, le mappature sono necessarie per le seguenti risorse, anche se possono essere necessarie mappature aggiuntive:

* Nodo principale della pagina del contenuto (sotto `/content`)
* Il nodo di progettazione utilizzato dalle pagine (sotto `/etc/designs`)
* `/libs` La cartella

Dopo aver creato la mappatura per la pagina di contenuto, per scoprire altre mappature necessarie utilizzate un browser Web per aprire una pagina sul server Web. Nel file error. log dell&#39;istanza di pubblicazione, individuare i messaggi sulle risorse non trovate. Il seguente messaggio di esempio indica che è necessaria `/etc/clientlibs` una mappatura per:

```shell
01.11.2012 15:59:24.601 *INFO* [10.36.34.243 [1351799964599] GET /etc/clientlibs/foundation/jquery.js HTTP/1.1] org.apache.sling.engine.impl.SlingRequestProcessorImpl service: Resource /content/sitea/etc/clientlibs/foundation/jquery.js not found
```

>[!NOTE]
>
>Il collegamento di collegamento del rewriter Apache Sling predefinito modifica automaticamente i collegamenti ipertestuali nella pagina per evitare collegamenti interrotti. Tuttavia, la riscrittura dei collegamenti viene eseguita solo quando la destinazione del collegamento è un file HTML o HTM. Per aggiornare i collegamenti ad altri tipi di file, create un componente transformer e aggiungetelo a una pipeline di reautori HTML.

### Esempio di nodi mappatura risorse

Nella tabella seguente sono elencati i nodi che implementano la mappatura delle risorse per il dominio branda.com. Per `brandb.com` il dominio vengono creati nodi simili, ad esempio `/etc/map/http/brandb.com`. In tutti i casi, le mappature sono necessarie quando i riferimenti nell&#39;HTML della pagina non vengono risolti correttamente nel contesto Sling.

| Percorso nodo | Tipo | Proprietà |
|--- |--- |--- |
| `/etc/map/http/branda.com` | sling: Mappatura | Nome: sling: Tipo di interdirect: Valore stringa: /content/sitea |
| `/etc/map/http/branda.com/libs` | sling: Mappatura | Name: sling:internalRedirect <br/>Type: String <br/>Value: /libs |
| `/etc/map/http/branda.com/etc` | sling: Mappatura |
| `/etc/map/http/branda.com/etc/designs` | sling: Mappatura | Nome: sling: Internalredirect <br/>vtype: Stringa <br/>vvalue: /etc/designs |
| `/etc/map/http/branda.com/etc/clientlibs` | sling: Mappatura | Nome: sling: Internalredirect <br/>vtype: Stringa <br/>vvalue: /etc/clientlibs |

## Configurazione dell&#39;agente di replica Svuotamento del dispatcher {#configuring-the-dispatcher-flush-replication-agent}

L&#39;agente replica Svuotamento del dispatcher nell&#39;istanza di pubblicazione AEM deve inviare richieste di annullamento validità alla fattoria corretta del dispatcher. Per eseguire il targeting di una fattoria, utilizzate la proprietà URI dell&#39;agente di replica Svuotamento del dispatcher (nella scheda Trasporto). Includete il valore della `/virtualhost` proprietà per la fattoria Dispatcher configurata per annullare la validità della cache:

`https://*webserver_name*:*port*/*virtual_host*/dispatcher/invalidate.cache`

Ad esempio, per utilizzare `farm_flush` la fattoria dell&#39;esempio precedente, l&#39;URI `https://localhost:80/invalidation_only/dispatcher/invalidate.cache`è.

![](assets/chlimage_1-12.png)

## Il server Web riscrive gli URL in entrata {#the-web-server-rewrites-incoming-urls}

Utilizzate la funzione di riscrittura URL interna del server Web per tradurre gli URL basati sul dominio in percorsi file nella cache del dispatcher. Ad esempio, le richieste del client per `https://brandA.com/en.html` la pagina vengono convertite nel `content/sitea/en.html`file nella directory principale del documento del server Web.

![](assets/chlimage_1-13.png)

La cache del dispatcher riflette la struttura del nodo archivio. Pertanto, quando si verificano delle attivazioni di pagina le richieste risultanti per annullare la pagina memorizzata nella cache non richiedono la traduzione di URL o percorsi.

![](assets/chlimage_1-14.png)

## Definire gli host virtuali e riscrivere regole sul server Web {#define-virtual-hosts-and-rewrite-rules-on-the-web-server}

Configurate gli aspetti seguenti sul server Web:

* Definire un host virtuale per ciascun dominio Web.
* Per ciascun dominio, configurate la radice del documento affinché coincida con la cartella nella directory archivio che contiene il contenuto Web del dominio.
* Per ciascun dominio virtuale, create una regola di ridenominazione URL che converte l&#39;URL in entrata nel percorso del file memorizzato nella cache.
* Ogni dominio virtuale deve includere anche configurazioni correlate al dispatcher, come descritto nella pagina [Installazione del dispatcher](dispatcher-install.md) .
* Il modulo Dispatcher deve essere configurato per utilizzare l&#39;URL che il server Web è stato riscritto. (Vedete `DispatcherUseProcessedURL` la proprietà in [Installazione del dispatcher](dispatcher-install.md).)

Il seguente file httpd. conf configura due host virtuali per un server Web Apache:

* I nomi dei server (che coincidono con i nomi di dominio) sono `brandA.com` (riga 16) e `brandB.com` (riga 32).

* La directory principale del documento di ogni dominio virtuale è la directory nella cache del dispatcher che contiene le pagine del sito. (righe 20 e 33)
* La regola di riscrittura URL per ogni dominio virtuale è un&#39;espressione regolare che prefissi il percorso della pagina richiesta con il percorso delle pagine nella cache. (righe 19 e 35)
* La `DispatherUseProcessedURL` proprietà è impostata su `1`. (riga 10)

Ad esempio, quando riceve una richiesta con l&#39; `https://brandA.com/en/products.html` URL, il server Web esegue le azioni seguenti:

* Associa l&#39;URL all&#39;host virtuale con `ServerName` un `brandA.com.`
* Riscrive l&#39;URL `/content/sitea/en/products.html.`
* Inoltrare l&#39;URL al dispatcher.

### httpd. conf {#httpd-conf-1}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 1
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/sitea/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/siteb/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

### Configurare una fattoria di Dispatcher {#configure-a-dispatcher-farm}

Quando il server Web riscrive gli URL, il dispatcher richiede una sola fattoria definita in [base alla configurazione del dispatcher](dispatcher-configuration.md). Per supportare gli ospitanti virtuali del server Web e le regole di ridenominazione URL, sono necessarie le configurazioni seguenti:

* La `/virtualhosts` proprietà deve includere i valori servername per tutte le definizioni virtualhost.
* La `/statfileslevel` proprietà deve essere sufficientemente alta per creare file. stat nelle directory contenenti i file di contenuto per ciascun dominio.

Il seguente file di configurazione di esempio si basa sul `dispatcher.any` file di esempio installato con Dispatcher. Per supportare le configurazioni server Web del file precedente `httpd.conf` sono necessarie le seguenti modifiche:

* La `/virtualhosts` proprietà causa la gestione delle richieste per i `brandA.com` domini da parte `brandB.com` del dispatcher. (riga 12)
* La `/statfileslevel` proprietà è impostata su 2, in modo che vengano creati file stat in ciascuna directory contenente il contenuto Web del dominio (riga 41): `/statfileslevel "2"`

Come al solito, il livello principale del documento della cache corrisponde alla radice del documento del server Web (riga 40): `/usr/lib/apache/httpd-2.4.3/htdocs`

### `dispatcher.any` {#dispatcher-any}

```xml
/name "testDispatcher"
/farms
  {
  /dispfarm0
    {  
    /clientheaders
      {
      "*"
      }      
    /virtualhosts
      {
      "brandA.com" "brandB.com"
      }
    /renders
      {
      /rend01    {  /hostname "127.0.0.1"   /port "4503"  }
      }
    /filter
      {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" }  # disable this rule to allow mapped content only
      /0041 { /type "allow" /glob "* *.css *"   }  # enable css
      /0042 { /type "allow" /glob "* *.gif *"   }  # enable gifs
      /0043 { /type "allow" /glob "* *.ico *"   }  # enable icos
      /0044 { /type "allow" /glob "* *.js *"    }  # enable javascript
      /0045 { /type "allow" /glob "* *.png *"   }  # enable png
      /0046 { /type "allow" /glob "* *.swf *"   }  # enable flash
      /0061 { /type "allow" /glob "POST /content/[.]*.form.html" }  # allow POSTs to form selectors under content
      /0062 { /type "allow" /glob "* /libs/cq/personalization/*"  }  # enable personalization
      /0081 { /type "deny"  /glob "GET *.infinity.json*" }
      /0082 { /type "deny"  /glob "GET *.tidy.json*"     }
      /0083 { /type "deny"  /glob "GET *.sysview.xml*"   }
      /0084 { /type "deny"  /glob "GET *.docview.json*"  }
      /0085 { /type "deny"  /glob "GET *.docview.xml*"  }      
      /0086 { /type "deny"  /glob "GET *.*[0-9].json*" }
      /0090 { /type "deny"  /glob "* *.query.json*" }
      }
    /cache
      {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
      /statfileslevel "2"
      /allowAuthorized "0"
      /rules
        {
        /0000  { /glob "*"     /type "allow"  }
        }
      /invalidate
        {
        /0000  {   /glob "*" /type "deny"  }
        /0001 {  /glob "*.html" /type "allow"  }
        }
      /allowedClients
        {
        }     
      }
    /statistics
      {
      /categories
        {
        /html  { /glob "*.html" }
        /others  {  /glob "*"  }
        }
      }
    }
  }
```

>[!NOTE]
>
>Poiché è definita una sola fattoria di Dispatcher, l&#39;agente di replica Pulizia del dispatcher nell&#39;istanza di pubblicazione AEM non richiede configurazioni speciali.

## Riscrittura di collegamenti in file non HTML {#rewriting-links-to-non-html-files}

Per riscrivere i riferimenti a file con estensioni diverse da.html o.htm, create un componente transformer Sling rewriter e aggiungetelo alla pipeline predefinita per i reautori.

Riscrivere i riferimenti quando i percorsi di risorse non vengono risolti correttamente nel contesto del server Web. Ad esempio, è necessario un transformer quando la generazione di immagini crea collegamenti come /content/sitea/en/products.navimage.png. Il componente superiore della [procedura Come creare un sito Web completo per Internet](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/the-basics.html) crea tali collegamenti.

Il [rewriter Sling](https://sling.apache.org/documentation/bundles/output-rewriting-pipelines-org-apache-sling-rewriter.html) è un modulo che elabora l&#39;output Sling. Le implementazioni della pipeline SAX del rewriter sono costituite da un generator, uno o più trasformatori e un serializzatore:

* **Generator:** Analizza il flusso di output Sling (documento HTML) e genera eventi SAX quando rileva tipi di elementi specifici.
* **Transformer:** Esegue l&#39;ascolto degli eventi SAX e, di conseguenza, modifica il target dell&#39;evento (un elemento HTML). Una pipeline di reautori contiene zero o più transformer. I trasformatori vengono eseguiti in sequenza, passando gli eventi SAX al passaggio successivo della sequenza.
* **Serializzatore:** Serializza l&#39;output, comprese le modifiche di ogni transformer.

![](assets/chlimage_1-15.png)

### Pipeline rewriter predefinita AEM {#the-aem-default-rewriter-pipeline}

AEM utilizza un editor di pipeline predefinito che elabora i documenti di tipo text/html:

* Il generator analizza i documenti HTML e genera eventi SAX quando rileva un&#39;area, un&#39;area, un&#39;area, un modulo, un elemento di base, uno script e elementi body. L&#39;alias generator è `htmlparser`.
* La pipeline include i seguenti trasformatori: `linkchecker``mobile``mobiledebug``contentsync`, Il `linkchecker` transformer esternalizza i percorsi per i file HTML o HTM di riferimento per evitare collegamenti interrotti.
* Il serializzatore scrive l&#39;output HTML. L&#39;alias serializer è htmlwriter.

Il `/libs/cq/config/rewriter/default` nodo definisce la pipeline.

### Creazione di un transformer {#creating-a-transformer}

Eseguite le seguenti attività per creare un componente transformer e utilizzarlo in una pipeline:

1. Implementare l&#39; `org.apache.sling.rewriter.TransformerFactory` interfaccia. Questa classe crea istanze della classe transformer. Specificate i valori per `transformer.type` la proprietà (l&#39;alias transformer) e configurate la classe come componente del servizio osgi.
1. Implementare l&#39; `org.apache.sling.rewriter.Transformer` interfaccia. Per ridurre al minimo il lavoro, potete estendere la `org.apache.cocoon.xml.sax.AbstractSAXPipe` classe. Sostituite il metodo startelement per personalizzare il comportamento di riscrittura. Questo metodo viene chiamato per ogni evento SAX passato al transformer.
1. Bundle e implementare le classi.
1. Aggiungete un nodo di configurazione all&#39;applicazione AEM per aggiungere il transformer alla pipeline.

>[!TIP]
>Potete invece configurare transformerfactory in modo che il transformer venga inserito in tutti i rewriter definiti. Di conseguenza, non è necessario configurare una pipeline:
>
>* Impostare `pipeline.mode` la proprietà su `global`.
>* Impostare `service.ranking` la proprietà su un numero intero positivo.
>* Non includete una `pipeline.type` proprietà.


>[!NOTE]
>
>Utilizzate l&#39;archetype [multimocanle](https://helpx.adobe.com/experience-manager/aem-previous-versions.html) del plug-in Content Package Maven per creare il progetto Maven. I POI creano e installano automaticamente un pacchetto di contenuto.

Gli esempi seguenti implementano un transformer che riscrive i riferimenti ai file immagine.

* La classe myrewritertransformerfactory crea un&#39;istanza degli oggetti myrewritertransformer. La proprietà pipeline. type imposta l&#39;alias transformer su mytransformer. Per includere l&#39;alias in una pipeline, il nodo di configurazione della pipeline include questo alias nell&#39;elenco dei trasformatori.
* La classe myrewritertransformer sostituisce il metodo startelement della classe abstractsaxtransformer. Il metodo startelement riscrive il valore degli attributi src per gli elementi img.

Gli esempi non sono affidabili e non devono essere utilizzati in un ambiente di produzione.

### Implementazione di un esempio transformerfactory {#example-transformerfactory-implementation}

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.rewriter.Transformer;
import org.apache.sling.rewriter.TransformerFactory;

@Component
@Service
public class MyRewriterTransformerFactory implements TransformerFactory {
    /* Define the alias */
    @Property(value="mytransformer")
    static final String PIPELINE_TYPE ="pipeline.type";
 
    public Transformer createTransformer() {
        
        return new MyRewriterTransformer ();
    }
}
```

### Esempio di implementazione transformer {#example-transformer-implementation}

```java
package com.adobe.example;

import java.io.IOException;

import org.apache.cocoon.xml.sax.AbstractSAXPipe;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.rewriter.ProcessingComponentConfiguration;
import org.apache.sling.rewriter.ProcessingContext;
import org.apache.sling.rewriter.Transformer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.AttributesImpl;

import javax.servlet.http.HttpServletRequest;

public class MyRewriterTransformer extends AbstractSAXPipe implements Transformer {

 private static final Logger log = LoggerFactory.getLogger(MyRewriterTransformer.class);
 private SlingHttpServletRequest httpRequest; 
 /* The element and attribute to act on  */
 private static final String ATT_NAME = new String("src");
 private static final String EL_NAME = new String("img");

 public MyRewriterTransformer () {
 }
 public void dispose() {
 }
 public void init(ProcessingContext context, ProcessingComponentConfiguration config) throws IOException {
  this.httpRequest = context.getRequest();
  log.debug("Transforming request {}.", httpRequest.getRequestURI());
 }
 @Override
 public void startElement (String nsUri, String localname, String qname, Attributes atts) throws SAXException {
  /* copy the element attributes */
  AttributesImpl linkAtts = new AttributesImpl(atts); 
  /* Only interested in EL_NAME elements */
  if(EL_NAME.equalsIgnoreCase(localname)){

   /* iterate through the attributes of the element and act only on ATT_NAME attributes */
   for (int i=0; i < linkAtts.getLength(); i++) {
    if (ATT_NAME.equalsIgnoreCase(linkAtts.getLocalName(i))) {
     String path_in_link = linkAtts.getValue(i);

     /* use the resource resolver of the http request to reverse-resolve the path  */
     String mappedPath = httpRequest.getResourceResolver().map(httpRequest, path_in_link);

     log.info("Tranformed {} to {}.", path_in_link,mappedPath);

     /* update the attribute value */
     linkAtts.setValue(i,mappedPath);
    }
   }

  }
        /* return updated attributes to super and continue with the transformer chain */
 super.startElement(nsUri, localname, qname, linkAtts);
 }
}
```

### Aggiunta di Transformer a una pipeline di reautori {#adding-the-transformer-to-a-rewriter-pipeline}

Crea un nodo JCR che definisca una pipeline che utilizza il tuo transformer. La seguente definizione node crea una pipeline che elabora file text/html. Viene utilizzato il generatore e il parser predefiniti AEM per l&#39;HTML.

>[!NOTE]
>
>Se impostate la proprietà Transformer `pipeline.mode` su `global`, non è necessario configurare una pipeline. La `global` modalità inserisce il transformer in tutti i pipeline.

### Nodo di configurazione rewriter - Rappresentazione XML {#rewriter-configuration-node-xml-representation}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="https://www.jcp.org/jcr/1.0" xmlns:nt="https://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    contentTypes="[text/html]"
    enabled="{Boolean}true"
    generatorType="htmlparser"
    order="5"
    serializerType="htmlwriter"
    transformerTypes="[mytransformer]">
</jcr:root>
```

L&#39;elemento grafico seguente mostra la rappresentazione CRXDE Lite del nodo:

![](assets/chlimage_1-16.png)
