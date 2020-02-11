---
title: 'Utilizzo di Dispatcher con più domini '
seo-title: 'Utilizzo di Dispatcher con più domini '
description: Scopri come utilizzare il dispatcher per elaborare le richieste di pagina in più domini Web.
seo-description: Scopri come utilizzare il dispatcher per elaborare le richieste di pagina in più domini Web.
uuid: 7342a1c2-fe61-49be-a240-b487d53c7ec1
contentOwner: User
cq-exporttemplate: /etc/contentsync/templates/geometrixx/page/rewrite
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 40d91d66-c99b-422d-8e61-c0ced23272ef
translation-type: tm+mt
source-git-commit: 64d26d802dbc9bb0b6815011a16e24c63a7672aa

---


# Utilizzo di Dispatcher con più domini{#using-dispatcher-with-multiple-domains}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potrebbe essere stato reindirizzato a questa pagina se avete seguito un collegamento alla documentazione del dispatcher incorporata nella documentazione di AEM o CQ.

Utilizzate Dispatcher per elaborare le richieste di pagina in più domini Web e al contempo supportare le seguenti condizioni:

* Il contenuto Web per entrambi i domini è memorizzato in un unico archivio AEM.
* I file nella cache del dispatcher possono essere invalidati separatamente per ciascun dominio.

Ad esempio, un&#39;azienda pubblica siti Web per due dei loro marchi: Marchio A e marchio B. Il contenuto delle pagine del sito Web viene creato in AEM e memorizzato nello stesso workspace del repository:

```
/
| - content  
   | - sitea  
       | - content nodes  
   | - siteb  
       | - content nodes
```

Le pagine per `BrandA.com` sono memorizzate di seguito `/content/sitea`. Le richieste client per l&#39;URL `https://BrandA.com/en.html` vengono restituite alla pagina di cui è stato effettuato il rendering per il `/content/sitea/en` nodo. Analogamente, le pagine per `BrandB.com` sono memorizzate di seguito `/content/siteb`.

Quando si utilizza Dispatcher per memorizzare nella cache il contenuto, è necessario creare associazioni tra l&#39;URL della pagina nella richiesta HTTP del client, il percorso del file corrispondente nella cache e il percorso del file corrispondente nell&#39;archivio.

## Richieste client

Quando i client inviano richieste HTTP al server Web, l’URL della pagina richiesta deve essere risolto nel contenuto presente nella cache del dispatcher e, infine, nel contenuto presente nell’archivio.

![](assets/chlimage_1-8.png)

1. Il sistema dei nomi di dominio rileva l’indirizzo IP del server Web registrato per il nome di dominio nella richiesta HTTP.
1. La richiesta HTTP viene inviata al server Web.
1. La richiesta HTTP viene passata al dispatcher.
1. Il dispatcher determina se i file memorizzati nella cache sono validi. Se validi, i file memorizzati nella cache vengono inviati al client.
1. Se i file memorizzati nella cache non sono validi, il dispatcher richiede le pagine appena sottoposte a rendering dall’istanza di pubblicazione AEM.

## Annullamento cache

Quando gli agenti di replica Dispatcher Flush richiedono che il dispatcher invalidi i file memorizzati nella cache, il percorso del contenuto nell&#39;archivio deve essere risolto nel contenuto della cache.

![](assets/chlimage_1-9.png)

1. Una pagina viene attivata nell’istanza di creazione di AEM e il contenuto viene replicato nell’istanza di pubblicazione.
1. Il Dispatcher Flush Agent chiama Dispatcher per annullare la validità della cache per il contenuto replicato.
1. Dispatcher tocca uno o più file .stat per annullare la validità dei file memorizzati nella cache.

Per utilizzare Dispatcher con più domini, è necessario configurare AEM, Dispatcher e il server Web. Le soluzioni descritte in questa pagina sono generali e si applicano alla maggior parte degli ambienti. A causa della complessità di alcune topologie di AEM, la soluzione può richiedere ulteriori configurazioni personalizzate per risolvere problemi particolari. Probabilmente dovrete adattare gli esempi per soddisfare le vostre attuali politiche di infrastruttura e gestione IT.

## Mappatura URL {#url-mapping}

Per abilitare gli URL di dominio e i percorsi di contenuto per la risoluzione dei file memorizzati nella cache, è necessario convertire in un determinato momento un percorso di file o un URL di pagina. Vengono fornite descrizioni delle seguenti strategie comuni, in cui le traduzioni di percorso o URL avvengono in punti diversi del processo:

* (consigliato) L’istanza di pubblicazione AEM utilizza la mappatura Sling per la risoluzione delle risorse per implementare le regole di riscrittura URL interne. Gli URL del dominio vengono convertiti in percorsi dell&#39;archivio dei contenuti. Consultate [AEM Riscrive gli URL](#aem-rewrites-incoming-urls)in arrivo.
* Il server Web utilizza regole interne di riscrittura URL che convertono gli URL del dominio nei percorsi della cache. Consultate [Il Server Web Riscrive Gli URL](#the-web-server-rewrites-incoming-urls)In Arrivo.

In genere è consigliabile utilizzare URL brevi per le pagine Web. In genere, gli URL delle pagine riflettono la struttura delle cartelle dell’archivio contenenti il contenuto Web. Tuttavia, gli URL non rivelano i nodi directory archivio principali, come `/content`. Il client non è necessariamente a conoscenza della struttura dell&#39;archivio AEM.

## Requisiti generali {#general-requirements}

Il tuo ambiente deve implementare le seguenti configurazioni per supportare il dispatcher che utilizza più domini:

* Il contenuto di ciascun dominio risiede in rami separati dell&#39;archivio (vedere l&#39;ambiente di esempio riportato di seguito).
* L&#39;agente di replica Dispatcher Flush è configurato nell&#39;istanza di pubblicazione AEM. (consultate [Invalidazione della cache del dispatcher da un&#39;istanza](page-invalidate.md)di pubblicazione.)
* Il sistema dei nomi di dominio risolve i nomi di dominio all&#39;indirizzo IP del server Web.
* La cache del dispatcher riflette la struttura di directory dell&#39;archivio dei contenuti di AEM. I percorsi dei file sotto la radice del documento del server Web sono gli stessi dei percorsi dei file nella directory archivio.

## Ambiente per gli esempi forniti {#environment-for-the-provided-examples}

Le soluzioni di esempio fornite si applicano a un ambiente con le seguenti caratteristiche:

* Le istanze di creazione e pubblicazione di AEM vengono distribuite sui sistemi Linux.
* Apache HTTPD è il server web, implementato su un sistema Linux.
* L’archivio dei contenuti di AEM e la directory principale del documento del server Web utilizzano le seguenti strutture di file (la directory principale del documento del server Web Apache è /`usr/lib/apache/httpd-2.4.3/htdocs)`:

   **Archivio**

```
  | - /content  
    | - sitea  
  |    | - content nodes 
    | - siteb  
       | - conent nodes
```

**Radice del documento del server Web**

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

La mappatura Sling per la risoluzione delle risorse consente di associare gli URL in arrivo ai percorsi di contenuto AEM. Create mappature sull’istanza di pubblicazione AEM in modo che le richieste di rendering dal dispatcher vengano indirizzate al contenuto corretto nell’archivio.

Le richieste del dispatcher per il rendering della pagina identificano la pagina utilizzando l’URL che viene trasmessa dal server Web. Quando l’URL include un nome di dominio, le mappature Sling risolvono l’URL del contenuto. L&#39;immagine seguente illustra una mappatura dell&#39; `branda.com/en.html` URL al `/content/sitea/en` nodo.

![](assets/chlimage_1-10.png)

La cache del Dispatcher riflette la struttura del nodo del repository. Pertanto, quando si verificano le attivazioni di pagina, le richieste risultanti per annullare la validità della pagina memorizzata nella cache non richiedono traduzioni di URL o percorsi.

![](assets/chlimage_1-11.png)

## Definire gli host virtuali sul server Web {#define-virtual-hosts-on-the-web-server}

Definite gli host virtuali sul server Web in modo che a ogni dominio Web possa essere assegnata una radice documento diversa:

* Il server Web deve definire un dominio virtuale per ciascun dominio Web.
* Per ciascun dominio, configurate la radice del documento in modo che coincida con la cartella nella directory archivio che contiene il contenuto Web del dominio.
* Ogni dominio virtuale deve includere anche configurazioni relative al dispatcher, come descritto nella pagina [Installazione del dispatcher](dispatcher-install.md) .

Il seguente `httpd.conf` file di esempio configura due domini virtuali per un server Web Apache:

* I nomi dei server (che coincidono con i nomi di dominio) sono branda.com (riga 16) e brandb.com (riga 30).
* La radice del documento di ciascun dominio virtuale è la directory nella cache del dispatcher che contiene le pagine del sito. (righe 17 e 31)

Con questa configurazione, il server Web esegue le azioni seguenti quando riceve una richiesta per `https://branda.com/en/products.html`:

* Associa l&#39;URL all&#39;host virtuale con un `ServerName` valore `branda.com.`

* Invia l’URL al dispatcher.

### httpd.conf {#httpd-conf}

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

Gli host virtuali ereditano il valore della proprietà [DispatcherConfig](dispatcher-install.md#main-pars-67-table-7) configurato nella sezione server principale. Gli host virtuali possono includere la propria proprietà DispatcherConfig per ignorare la configurazione del server principale.

### Configurare il dispatcher per gestire più domini {#configure-dispatcher-to-handle-multiple-domains}

Per supportare gli URL che includono nomi di dominio e host virtuali corrispondenti, definire le seguenti farm di dispatcher:

* Configurare una farm del dispatcher per ciascun host virtuale. Queste farm elaborano le richieste dal server Web per ciascun dominio, verificano la presenza di file memorizzati nella cache e richiedono le pagine dai rendering.
* Configurate una farm di dispatcher utilizzata per annullare la validità del contenuto nella cache, indipendentemente dal dominio a cui appartiene il contenuto. Questa farm gestisce le richieste di annullamento della convalida dei file da parte degli agenti di replica del dispatcher di Flash.

### Creazione di farm di dispatcher per gli host virtuali

Le farm per gli host virtuali devono avere le seguenti configurazioni in modo che gli URL nelle richieste HTTP client vengano risolti nei file corretti nella cache del dispatcher:

* La `/virtualhosts` proprietà è impostata sul nome del dominio. Questa proprietà consente al Dispatcher di associare la farm al dominio.
* La `/filter` proprietà consente di accedere al percorso dell’URL della richiesta troncato dopo la parte relativa al nome di dominio. Ad esempio, per l’ `https://branda.com/en.html` URL, il percorso viene interpretato come `/en.html`, pertanto il filtro deve consentire l’accesso a questo percorso.

* La `/docroot` proprietà è impostata sul percorso della directory principale del contenuto del sito del dominio nella cache del dispatcher. Questo percorso viene utilizzato come prefisso per l’URL concatenato della richiesta originale. Ad esempio, il docroot di `/usr/lib/apache/httpd-2.4.3/htdocs/sitea` causa la richiesta di `https://branda.com/en.html` risoluzione al `/usr/lib/apache/httpd-2.4.3/htdocs/sitea/en.html` file.

Inoltre, l’istanza di pubblicazione AEM deve essere designata come rendering per l’host virtuale. Configura le altre proprietà della farm come richiesto. Il codice seguente è una configurazione farm abbreviata per il dominio branda.com:

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

### Creazione di una farm di dispatcher per l&#39;annullamento della validità della cache

Per gestire le richieste di annullamento della validità dei file memorizzati nella cache è necessaria una farm di dispatcher. Questa farm deve essere in grado di accedere ai file .stat nelle directory docroot di ciascun host virtuale.

Le seguenti configurazioni di proprietà consentono al Dispatcher di risolvere i file presenti nell’archivio dei contenuti di AEM dai file presenti nella cache:

* La `/docroot` proprietà è impostata sul docroot predefinito del server Web. In genere si tratta della directory in cui viene creata la `/content` cartella. Un valore di esempio per Apache su Linux è `/usr/lib/apache/httpd-2.4.3/htdocs`.
* La `/filter` proprietà consente l&#39;accesso ai file sotto la `/content` directory.

La `/statfileslevel`proprietà deve essere sufficientemente alta in modo che i file .stat vengano creati nella directory principale di ciascun host virtuale. Questa proprietà consente di annullare la validità della cache di ciascun dominio separatamente. Per l&#39;impostazione di esempio, un `/statfileslevel` valore di `2` crea file .stat nella `*docroot*/content/sitea` directory e nella `*docroot*/content/siteb` directory.

Inoltre, l’istanza di pubblicazione deve essere designata come rendering per l’host virtuale. Configura le altre proprietà della farm come richiesto. Il codice seguente è una configurazione abbreviata per la farm utilizzata per annullare la validità della cache:

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

Quando si avvia il server Web, il registro del dispatcher (in modalità di debug) indica l&#39;inizializzazione di tutte le farm:

```shell
Dispatcher initializing (build 4.1.2)
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_sitea].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_siteb].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_flush].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs
[Fri Nov 02 16:27:18 2012] [I] [24974(140006182991616)] Dispatcher initialized (build 4.1.2)
```

### Configurare il mapping Sling per la risoluzione delle risorse {#configure-sling-mapping-for-resource-resolution}

Utilizzate la mappatura Sling per la risoluzione delle risorse in modo che gli URL basati sul dominio risolvano il contenuto nell’istanza di pubblicazione AEM. La mappatura delle risorse converte gli URL in entrata dal dispatcher (originariamente dalle richieste HTTP client) ai nodi di contenuto.

Per ulteriori informazioni sulla mappatura delle risorse Sling, consulta [Mappature per la risoluzione](https://sling.apache.org/site/mappings-for-resource-resolution.html) delle risorse nella documentazione Sling.

In genere, le mappature sono necessarie per le risorse seguenti, anche se possono essere necessarie mappature aggiuntive:

* Il nodo principale della pagina del contenuto (sotto `/content`)
* Il nodo di progettazione utilizzato dalle pagine (sotto `/etc/designs`)
* La `/libs` cartella

Dopo aver creato la mappatura per la pagina del contenuto, per scoprire le mappature aggiuntive richieste utilizzate un browser Web per aprire una pagina sul server Web. Nel file error.log dell’istanza di pubblicazione, individuate i messaggi relativi alle risorse che non sono stati trovati. Il seguente messaggio di esempio indica che `/etc/clientlibs` è necessaria una mappatura:

```shell
01.11.2012 15:59:24.601 *INFO* [10.36.34.243 [1351799964599] GET /etc/clientlibs/foundation/jquery.js HTTP/1.1] org.apache.sling.engine.impl.SlingRequestProcessorImpl service: Resource /content/sitea/etc/clientlibs/foundation/jquery.js not found
```

>[!NOTE]
>
>Il trasformatore linkchecker della riscrittura Apache Sling predefinita modifica automaticamente i collegamenti ipertestuali nella pagina per evitare collegamenti interrotti. Tuttavia, la riscrittura del collegamento viene eseguita solo quando la destinazione del collegamento è un file HTML o HTM. Per aggiornare i collegamenti ad altri tipi di file, create un componente trasformatore e aggiungetelo a una pipeline di riscrittura HTML.

### Esempio di nodi di mappatura delle risorse

Nella tabella seguente sono elencati i nodi che implementano la mappatura delle risorse per il dominio branda.com. Nodi simili vengono creati per il `brandb.com` dominio, ad esempio `/etc/map/http/brandb.com`. In tutti i casi, le mappature sono necessarie quando i riferimenti nella pagina HTML non vengono risolti correttamente nel contesto di Sling.

| Percorso nodo | Tipo | Proprietà |
|--- |--- |--- |
| `/etc/map/http/branda.com` | sling:mapping | Nome: sling:internalRedirect Type: Valore stringa: /content/sitea |
| `/etc/map/http/branda.com/libs` | sling:mapping | Nome: sling:internalRedirect <br/>Type: Valore <br/>stringa: /libs |
| `/etc/map/http/branda.com/etc` | sling:mapping |  |
| `/etc/map/http/branda.com/etc/designs` | sling:mapping | Nome: sling:internalRedirect <br/>VType: Valore <br/>stringa: /etc/designs |
| `/etc/map/http/branda.com/etc/clientlibs` | sling:mapping | Nome: sling:internalRedirect <br/>VType: Valore <br/>stringa: /etc/clientlibs |

## Configurazione dell&#39;agente di replica Dispatcher Flush {#configuring-the-dispatcher-flush-replication-agent}

L&#39;agente di replica Dispatcher Flush nell&#39;istanza di pubblicazione AEM deve inviare richieste di annullamento della validità alla farm del dispatcher corretta. Per eseguire il targeting di una farm, utilizzare la proprietà URI dell&#39;agente di replica Dispatcher Flush (nella scheda Trasporto). Includete il valore della `/virtualhost` proprietà per la farm del dispatcher configurata per annullare la validità della cache:

`https://*webserver_name*:*port*/*virtual_host*/dispatcher/invalidate.cache`

Ad esempio, per utilizzare la `farm_flush` farm dell&#39;esempio precedente, l&#39;URI è `https://localhost:80/invalidation_only/dispatcher/invalidate.cache`.

![](assets/chlimage_1-12.png)

## Il server Web riscrive gli URL in arrivo {#the-web-server-rewrites-incoming-urls}

Utilizzate la funzione di riscrittura URL interna del server Web per tradurre gli URL basati sul dominio in percorsi di file nella cache del dispatcher. Ad esempio, le richieste client per la `https://brandA.com/en.html` pagina vengono convertite nel `content/sitea/en.html`file nella directory principale del documento del server Web.

![](assets/chlimage_1-13.png)

La cache del Dispatcher riflette la struttura del nodo del repository. Pertanto, quando si verificano le attivazioni di pagina, le richieste risultanti per annullare la validità della pagina memorizzata nella cache non richiedono traduzioni di URL o percorsi.

![](assets/chlimage_1-14.png)

## Definire host virtuali e riscrivere le regole sul server Web {#define-virtual-hosts-and-rewrite-rules-on-the-web-server}

Configurate i seguenti aspetti sul server Web:

* Definite un host virtuale per ciascuno dei vostri domini Web.
* Per ciascun dominio, configurate la radice del documento in modo che coincida con la cartella nella directory archivio che contiene il contenuto Web del dominio.
* Per ciascun dominio virtuale, create una regola di ridenominazione URL che converte l’URL in arrivo nel percorso del file memorizzato nella cache.
* Ogni dominio virtuale deve includere anche configurazioni relative al dispatcher, come descritto nella pagina [Installazione del dispatcher](dispatcher-install.md) .
* Il modulo Dispatcher deve essere configurato per utilizzare l&#39;URL riscritto dal server Web. (vedere la `DispatcherUseProcessedURL` proprietà in [Installazione del dispatcher](dispatcher-install.md).)

Il seguente file httpd.conf di esempio configura due host virtuali per un server Web Apache:

* I nomi dei server (che coincidono con i nomi di dominio) sono `brandA.com` (riga 16) e `brandB.com` (riga 32).

* La radice del documento di ciascun dominio virtuale è la directory nella cache del dispatcher che contiene le pagine del sito. (linee 20 e 33)
* La regola di riscrittura URL per ciascun dominio virtuale è un&#39;espressione regolare che prefissa il percorso della pagina richiesta con il percorso delle pagine nella cache. (righe 19 e 35)
* La `DispatherUseProcessedURL` proprietà è impostata su `1`. (riga 10)

Ad esempio, quando riceve una richiesta con l’ `https://brandA.com/en/products.html` URL, il server Web esegue le azioni seguenti:

* Associa l&#39;URL all&#39;host virtuale con un `ServerName` valore `brandA.com.`
* Riscrive l’URL da assegnare `/content/sitea/en/products.html.`
* Invia l’URL al dispatcher.

### httpd.conf {#httpd-conf-1}

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

### Configurare una farm di dispatcher {#configure-a-dispatcher-farm}

Quando il server Web riscrive gli URL, il dispatcher richiede una singola farm definita in base alla [configurazione del dispatcher](dispatcher-configuration.md). Per supportare gli host virtuali del server Web e le regole di ridenominazione URL sono necessarie le seguenti configurazioni:

* La `/virtualhosts` proprietà deve includere i valori ServerName per tutte le definizioni VirtualHost.
* La `/statfileslevel` proprietà deve essere sufficientemente alta per creare file .stat nelle directory che contengono i file di contenuto per ciascun dominio.

Il seguente file di configurazione di esempio è basato sul `dispatcher.any` file di esempio installato con Dispatcher. Per supportare le configurazioni del server Web del `httpd.conf` file precedente sono necessarie le seguenti modifiche:

* La `/virtualhosts` proprietà fa sì che il dispatcher gestisca le richieste per i `brandA.com` domini e `brandB.com` . (riga 12)
* La `/statfileslevel` proprietà è impostata su 2, in modo che i file di stato vengano creati in ogni directory che contiene il contenuto Web del dominio (riga 41): `/statfileslevel "2"`

Come al solito, la radice del documento della cache è la stessa della radice del documento del server Web (riga 40): `/usr/lib/apache/httpd-2.4.3/htdocs`

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
>Poiché è definita una singola farm Dispatcher, l&#39;agente di replica Dispatcher Flush nell&#39;istanza di pubblicazione AEM non richiede configurazioni speciali.

## Riscrittura dei collegamenti a file non HTML {#rewriting-links-to-non-html-files}

Per riscrivere i riferimenti a file con estensioni diverse da .html o .htm, create un componente trasformatore di riscrittura Sling e aggiungetelo alla pipeline di riscrittura predefinita.

Riscrittura dei riferimenti quando i percorsi delle risorse non vengono risolti correttamente nel contesto del server Web. Ad esempio, è necessario un trasformatore quando componenti che generano immagini creano collegamenti come /content/sitea/en/products.navimage.png. Il componente Navigazione principale di [Come creare un sito Web](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/the-basics.html) Internet completo crea tali collegamenti.

La riscrittura [Sling](https://sling.apache.org/documentation/bundles/output-rewriting-pipelines-org-apache-sling-rewriter.html) è un modulo che postula l&#39;elaborazione dell&#39;output Sling. Le implementazioni della pipeline SAX della riscrittura sono costituite da un generatore, uno o più trasformatori e un serializzatore:

* **** Generatore: Analizza il flusso di output Sling (documento HTML) e genera eventi SAX quando incontra tipi di elementi specifici.
* **** Trasformatore: Elenca gli eventi SAX e modifica di conseguenza la destinazione dell&#39;evento (un elemento HTML). Una pipeline di riscrittura contiene zero o più trasformatori. I trasformatori vengono eseguiti in sequenza, passando gli eventi SAX al trasformatore successivo nella sequenza.
* **** Serializzatore: Serializza l&#39;output, comprese le modifiche apportate a ciascun trasformatore.

![](assets/chlimage_1-15.png)

### La pipeline di riscrittura predefinita di AEM {#the-aem-default-rewriter-pipeline}

AEM utilizza un rewriter di pipeline predefinito che elabora documenti di tipo text/html:

* Il generatore analizza i documenti HTML e genera eventi SAX quando incontra un elemento, img, area, modulo, base, collegamento, script e corpo. L&#39;alias del generatore è `htmlparser`.
* La pipeline include i seguenti trasformatori: `linkchecker`, `mobile`, `mobiledebug`, `contentsync`. Il `linkchecker` trasformatore esternalizza i percorsi ai file HTML o HTM di riferimento per evitare collegamenti interrotti.
* Il serializzatore scrive l&#39;output HTML. L&#39;alias del serializzatore è htmlwriter.

Il `/libs/cq/config/rewriter/default` nodo definisce la pipeline.

### Creazione di un trasformatore {#creating-a-transformer}

Per creare un componente trasformatore e utilizzarlo in una pipeline, effettuate le seguenti operazioni:

1. Implementare l&#39; `org.apache.sling.rewriter.TransformerFactory` interfaccia. Questa classe crea istanze della classe di trasformatori. Specificate i valori per la `transformer.type` proprietà (l’alias del trasformatore) e configurate la classe come componente del servizio OSGi.
1. Implementare l&#39; `org.apache.sling.rewriter.Transformer` interfaccia. Per ridurre al minimo il lavoro, è possibile estendere la `org.apache.cocoon.xml.sax.AbstractSAXPipe` classe. Ignorate il metodo startElement per personalizzare il comportamento di riscrittura. Questo metodo viene chiamato per ogni evento SAX passato al trasformatore.
1. Eseguire il bundle e distribuire le classi.
1. Aggiungi un nodo di configurazione all’applicazione AEM per aggiungere il trasformatore alla pipeline.

>[!TIP]
>È invece possibile configurare TransformerFactory in modo che il trasformatore venga inserito in ogni rewriter definito. Di conseguenza non è necessario configurare una pipeline:
>
>* Impostare la `pipeline.mode` proprietà su `global`.
>* Impostare la `service.ranking` proprietà su un numero intero positivo.
>* Non includere una `pipeline.type` proprietà.


>[!NOTE]
>
>Utilizzate l&#39;archetipo [multimodulo](https://helpx.adobe.com/experience-manager/aem-previous-versions.html) del plug-in Content Package Maven per creare il vostro progetto Maven. I POM creano e installano automaticamente un pacchetto di contenuti.

Gli esempi seguenti implementano un trasformatore che riscrive i riferimenti ai file immagine.

* La classe MyRewriterTransformerFactory crea un&#39;istanza degli oggetti MyRewriterTransformer. La proprietà pipeline.type imposta l&#39;alias del trasformatore su mytransformer. Per includere l&#39;alias in una pipeline, il nodo di configurazione della pipeline include questo alias nell&#39;elenco dei trasformatori.
* La classe MyRewriterTransformer sostituisce il metodo startElement della classe AbstractSAXTransformer. Il metodo startElement riscrive il valore degli attributi src per gli elementi img.

Gli esempi non sono robusti e non devono essere utilizzati in un ambiente di produzione.

### Esempio di implementazione di TransformerFactory {#example-transformerfactory-implementation}

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

### Esempio di implementazione del trasformatore {#example-transformer-implementation}

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

### Aggiunta del trasformatore a una pipeline di riscrittura {#adding-the-transformer-to-a-rewriter-pipeline}

Creare un nodo JCR che definisca una pipeline che utilizza il trasformatore. La seguente definizione del nodo crea una pipeline che elabora i file text/html. Vengono utilizzati il generatore AEM e il parser predefiniti per HTML.

>[!NOTE]
>
>Se si imposta la proprietà Transformer `pipeline.mode` su `global`, non è necessario configurare una pipeline. La `global` modalità inserisce il trasformatore in tutte le tubazioni.

### Nodo di configurazione della riscrittura - Rappresentazione XML {#rewriter-configuration-node-xml-representation}

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

L&#39;immagine seguente mostra la rappresentazione CRXDE Lite del nodo:

![](assets/chlimage_1-16.png)
