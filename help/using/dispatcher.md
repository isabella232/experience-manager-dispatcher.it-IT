---
title: Panoramica di Dispatcher
seo-title: Adobe AEM Dispatcher Overview
description: Scopri come utilizzare Dispatcher per migliorare la sicurezza, la memorizzazione in cache e altro ancora in AEM Cloud Services.
seo-description: This article provides a general overview of Adobe Experience Manager Dispatcher.
uuid: 71766f86-5e91-446b-a078-061b179d090d
pageversionid: 1193211344162
topic-tags: dispatcher
content-type: reference
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
exl-id: c9266683-6890-4359-96db-054b7e856dd0
source-git-commit: e87af532ee3268f0a45679e20031c3febc02de58
workflow-type: tm+mt
source-wordcount: '3165'
ht-degree: 57%

---

# Panoramica di Dispatcher {#dispatcher-overview}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

Dispatcher è uno strumento di caching e bilanciamento del carico di Adobe Experience Manager utilizzato con un server web di classe enterprise.

Il processo di distribuzione di Dispatcher è indipendente dal server web e dalla piattaforma del sistema operativo scelti:

1. Scopri di più su Dispatcher (questa pagina). Vedi anche [domande frequenti su Dispatcher](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/troubleshooting/dispatcher-faq.html?lang=en).
1. Installa un [server web supportato](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=it) in base alla documentazione di quel server web.
1. [Installa il modulo Dispatcher](dispatcher-install.md) nel server web e configura di conseguenza il server web.
1. [Configura Dispatcher](dispatcher-configuration.md) (il file dispatcher.any).
1. [Configura AEM](page-invalidate.md) in modo che gli aggiornamenti del contenuto invalidino la cache.

>[!NOTE]
>
>Per capire meglio come funziona Dispatcher con AEM:
>
>* Vedi [Chiedi agli esperti della community AEM di luglio 2017](https://communities.adobeconnect.com/pf0gem7igw1f/).
>* Accedi a [questo archivio](https://github.com/adobe/aem-dispatcher-experiments). Contiene una raccolta di esperimenti in un formato di laboratorio &quot;take-home&quot;.



Utilizza le seguenti informazioni come richiesto:

* [Elenco di controllo della sicurezza di Dispatcher](security-checklist.md)
* [Knowledge Base di Dispatcher](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html)
* [Ottimizzazione delle prestazioni della cache di un sito web](https://experienceleague.adobe.com/docs/experience-manager-64/deploying/configuring/configuring-performance.html?lang=en)
* [Utilizzo di Dispatcher con più domini](dispatcher-domains.md)
* [Utilizzo di SSL con Dispatcher](dispatcher-ssl.md)
* [Implementazione del caching sensibile alle autorizzazioni](permissions-cache.md)
* [Risoluzione dei problemi di Dispatcher](dispatcher-troubleshooting.md)
* [Domande frequenti sui principali problemi di Dispatcher](dispatcher-faq.md)

>[!NOTE]
>
>**Dispatcher viene utilizzato in genere** per memorizzare in cache le risposte di un’**istanza AEM Publish**, per aumentare la capacità di risposta e la sicurezza del sito web pubblicato che interfaccia con l’esterno. La maggior parte della discussione è incentrata su questo caso specifico.
>
>È però anche possibile utilizzare Dispatcher per incrementare la reattività dell’**istanza Autore**, in particolare se il sito web viene modificato e aggiornato da un numero elevato di utenti. Per i dettagli specifici di questo caso, vedi [Utilizzo di Dispatcher con Author Server](#using-a-dispatcher-with-an-author-server) più avanti.

## Perché usare Dispatcher per implementare il caching? {#why-use-dispatcher-to-implement-caching}

Sono due gli approcci di base alla pubblicazione sul web:

* **Server web statici**: come Apache o IIS, sono semplici, ma veloci.
* **Server di gestione del contenuto**: che forniscono contenuto intelligente, dinamico e in tempo reale, ma richiedono un tempo di calcolo maggiore e altre risorse.

Dispatcher permette di realizzare un ambiente al tempo stesso veloce e dinamico. Viene utilizzato parte come parte un server HTML statico, ad esempio Apache, con lo scopo di:

* Archiviare (o “memorizzare in cache”) la maggior parte del contenuto del sito sotto forma di sito web statico
* Accedere il meno possibile al motore di layout.

Questo significa che:

* **contenuto statico** viene gestito con la stessa velocità e facilità di un server web statico. Inoltre, puoi utilizzare gli strumenti di amministrazione e sicurezza disponibili per i server web statici.

* Il **contenuto dinamico** viene generato in base alle necessità, senza rallentare il sistema più del necessario.

Dispatcher contiene meccanismi per generare e aggiornare il codice HTML statico in base al contenuto del sito dinamico. È possibile specificare in dettaglio quali documenti vengono archiviati come file statici e quali vengono sempre generati in modo dinamico.

Questa sezione illustra i principi alla base di questo processo.

### Server web statico {#static-web-server}

![](assets/chlimage_1-3.png)

Un server web statico, come Apache o IIS, viene utilizzato per distribuire file HTML statici ai visitatori del sito web. Le pagine statiche vengono create una volta, quindi lo stesso contenuto viene distribuito per ogni richiesta.

Questo processo è semplice ed efficiente. Se un visitatore richiede un file come una pagina HTML, il file viene prelevato direttamente dalla memoria; nel peggiore dei casi, viene letto dall&#39;unità locale. I server web statici sono disponibili da molto tempo, per cui esiste un&#39;ampia gamma di strumenti per l&#39;amministrazione e la gestione della sicurezza e sono perfettamente integrati con le infrastrutture di rete.

### Server di gestione dei contenuti {#content-management-servers}

![](assets/chlimage_1-4.png)

Se utilizzi un CMS (Content Management Server), ad esempio AEM, la richiesta di un visitatore viene elaborata da un motore di layout avanzato. Il motore legge il contenuto da un archivio che, combinato con stili, formati e diritti di accesso, trasforma il contenuto in un documento personalizzato in base alle esigenze e ai diritti di un visitatore.

Questo flusso di lavoro consente di creare contenuti più ricchi e dinamici, che aumentano la flessibilità e le funzionalità del sito web. Tuttavia, il motore di layout richiede una potenza di elaborazione maggiore rispetto a un server statico, pertanto questa configurazione potrebbe essere soggetta a rallentamenti, se molti visitatori utilizzano il sistema.

## Funzionamento del caching in Dispatcher {#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**Directory della cache** Per il caching il modulo Dispatcher utilizza la funzionalità del server web che consente di distribuire contenuto statico. Dispatcher inserisce i documenti memorizzati in cache nella directory principale dei documenti del server web.

>[!NOTE]
>
>Se manca la configurazione per il caching delle intestazioni HTTP, Dispatcher archivia solo il codice HTML della pagina e non le intestazioni HTTP. Questo scenario può essere un problema se utilizzi codifiche diverse all’interno del sito web, in quanto queste pagine potrebbero andare perse. Per abilitare il caching delle intestazioni HTTP, vedi [Configurazione della cache di Dispatcher.](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/dispatcher-configuration.html?lang=it)

>[!NOTE]
>
>Se la directory principale dei documenti del server web si trova su Network Attached Storage (NAS), potresti riscontrare un degrado delle prestazioni. Inoltre, quando una directory principale dei documenti sul NAS viene condivisa tra più server web, possono verificarsi blocchi intermittenti quando vengono eseguite le azioni di replica.

>[!NOTE]
>
>Dispatcher archivia il documento memorizzato in cache in una struttura uguale all’URL richiesto.
>
>Possono esserci limitazioni a livello di sistema operativo per la lunghezza del nome del file. Cioè, se hai un URL con numerosi selettori.

### Metodi di caching

In Dispatcher sono disponibili due metodi principali per aggiornare il contenuto della cache quando vengono apportate modifiche al sito web.

* **Aggiornamenti del contenuto** rimuovi le pagine modificate e i file direttamente associati ad esse.
* **Annullamento automatico della validità**: le parti della cache che potrebbero risultare obsolete dopo un aggiornamento vengono invalidate automaticamente. In altre parole, contrassegna efficacemente le pagine rilevanti come non aggiornate, senza eliminare nulla.

### Aggiornamenti del contenuto

In un aggiornamento del contenuto uno o più documenti AEM cambiano. AEM invia una richiesta di distribuzione del contenuto (syndication) a Dispatcher, che aggiorna la cache di conseguenza:

1. Elimina i file modificati dalla cache.
1. Elimina dalla cache tutti i file che iniziano con lo stesso handle. Ad esempio, se viene aggiornato il file /en/index.html, tutti i file che iniziano con “/en/index” vengono eliminati. Questo meccanismo consente di progettare siti efficienti in termini di cache, soprattutto per la navigazione delle immagini.
1. It *tocco* il cosiddetto **statfile**, che aggiorna la marca temporale dello statfile per indicare la data dell’ultima modifica.

Tieni presente le seguenti considerazioni:

* Gli aggiornamenti dei contenuti vengono in genere utilizzati con un sistema di authoring che &quot;sa&quot; cosa deve essere sostituito.
* I file interessati da un aggiornamento del contenuto vengono rimossi, ma non sostituiti immediatamente. Alla successiva richiesta di tale file, Dispatcher recupera il nuovo file dall’istanza AEM e lo inserisce nella cache, sovrascrivendo il vecchio contenuto.
* In genere, le immagini generate automaticamente che incorporano testo di una pagina vengono archiviate in file di immagine che iniziano con lo stesso handle, in modo da garantire l’associazione per l’eliminazione. Ad esempio, puoi archiviare il testo del titolo della pagina mypage.html come immagine mypage.titlePicture.gif nella stessa cartella. In questo modo l’immagine viene eliminata automaticamente dalla cache ogni volta che la pagina viene aggiornata e potrai essere sicuro che l’immagine rispecchierà sempre la versione corrente della pagina.
* Possono esistere diversi statfile, ad esempio uno per cartella della lingua. Se una pagina viene aggiornata, AEM cerca la cartella padre successiva contenente uno statfile e *tocca* tale file.

### Annullamento automatico della validità

Con l’annullamento automatico della validità, parti della cache vengono automaticamente invalidate, ma non viene fisicamente eliminato alcun file. A ogni aggiornamento del contenuto, il cosiddetto statfile viene toccato e la marca temporale corrispondente rispecchia quindi l’ultimo aggiornamento del contenuto.

In Dispatcher è disponibile un elenco di file soggetti ad annullamento automatico della validità. Quando viene richiesto un documento da tale elenco, Dispatcher confronta la data del documento memorizzato in cache con la marca temporale dello statfile:

* Se il documento memorizzato in cache è più recente, Dispatcher lo restituisce.
* Se è precedente, Dispatcher recupera la versione corrente dall’istanza di AEM.

Anche in questo caso, tieni presente le seguenti considerazioni:

* L’annullamento automatico della validità viene in genere utilizzato quando le interrelazioni sono complesse, ad esempio nelle pagine di HTML. Queste pagine contengono collegamenti e voci di navigazione e in genere devono essere aggiornate dopo un aggiornamento del contenuto. Se sono stati generati automaticamente file PDF o immagine, è possibile scegliere di annullare automaticamente la validità anche di tali file.
* L’annullamento automatico della validità non richiede alcuna azione da parte del Dispatcher al momento dell’aggiornamento, ad eccezione del tocco sullo statfile. Tuttavia, quando lo statfile viene toccato, il contenuto della cache diventa automaticamente obsoleto, anche se non viene rimosso fisicamente dalla cache.

## Restituzione dei documenti in Dispatcher {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### Stabilire se un documento è soggetto a caching

Nel file di configurazione puoi [definire quali documenti Dispatcher deve memorizzare in cache](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/dispatcher-configuration.html?lang=it). Dispatcher confronta la richiesta con l’elenco dei documenti memorizzabili in cache. Se il documento non è incluso in questo elenco, Dispatcher richiede il documento dall’istanza di AEM.

Dispatcher richiede sempre il documento direttamente dall’istanza di AEM nei seguenti casi:

* L&#39;URI della richiesta contiene un punto interrogativo &quot;`?`&quot;. Questo scenario in genere indica una pagina dinamica, ad esempio un risultato di ricerca, che non deve essere memorizzata nella cache.
* Se manca l’estensione del file. Il server web ha bisogno dell’estensione per determinare il tipo di documento (tipo MIME).
* L&#39;intestazione di autenticazione è impostata (configurabile).

>[!NOTE]
>
>Dispatcher può memorizzare in cache i metodi GET o HEAD (per l’intestazione HTTP). Per ulteriori informazioni sul caching delle intestazioni di risposta, vedi la sezione [Caching delle intestazioni di risposta HTTP](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/dispatcher-configuration.html?lang=it).

### Stabilire se un documento è memorizzato in cache

Dispatcher archivia nel server web i file memorizzati in cache come se facessero parte di un sito web statico. Se un utente richiede un documento memorizzabile in cache, Dispatcher controlla se tale documento esiste nel file system del server Web:

* Se il documento è memorizzato in cache, Dispatcher restituisce il file.
* Se non è memorizzato in cache, Dispatcher richiede il documento dall’istanza di AEM.

### Determinare se un documento è aggiornato

Per verificare se un documento è aggiornato, Dispatcher esegue due passaggi:

1. Verifica se il documento è soggetto ad annullamento automatico della validità. Se non lo è, il documento viene considerato aggiornato.
1. Se il documento è configurato per l’annullamento automatico della validità, Dispatcher controlla se è più o meno recente dell’ultima modifica disponibile. Se è meno recente, Dispatcher richiede la versione corrente dall’istanza di AEM e sostituisce la versione nella cache.

>[!NOTE]
>
>Documenti senza **annullamento automatico della validità** rimangono nella cache fino a quando non vengono fisicamente eliminate. Ad esempio, tramite un aggiornamento del contenuto del sito Web.

## Vantaggi del bilanciamento del carico {#the-benefits-of-load-balancing}

Per bilanciamento del carico si intende la distribuzione del carico di elaborazione del sito web tra più istanze di AEM.

![](assets/chlimage_1-7.png)

I vantaggi di questa pratica sono i seguenti:

* **maggiore potenza di elaborazione**
In pratica, una maggiore potenza di elaborazione significa che Dispatcher condivide le richieste di documenti tra più istanze di AEM. Poiché il numero di documenti da elaborare per ogni istanza è ridotto, i tempi di risposta risultano inferiori. Dispatcher mantiene statistiche interne per ogni categoria di documenti, in modo da poter stimare il carico e distribuire le query in modo efficiente.

* **maggiore copertura in caso di errore**
Se Dispatcher non riceve risposte da un’istanza, invia automaticamente le richieste a una delle altre istanze. Se un&#39;istanza non è più disponibile, l&#39;unico effetto è un rallentamento del sito, proporzionato alla potenza di calcolo persa. Tuttavia, tutti i servizi continuano.

* È inoltre possibile gestire siti web diversi sullo stesso server web statico.

>[!NOTE]
>
>Mentre il bilanciamento del carico consente di distribuire il carico in modo efficiente, la memorizzazione in cache contribuisce a ridurlo. Prova quindi a ottimizzare il caching e ridurre il carico complessivo prima di configurare il bilanciamento del carico. Un caching appropriato può contribuire a migliorare le prestazioni del load balancer o rendere superfluo il bilanciamento del carico.

>[!CAUTION]
>
>Anche se un’unica istanza di Dispatcher è in grado di assorbire la capacità delle istanze di pubblicazione disponibili, per alcune rare applicazioni può essere utile bilanciare anche il carico tra due istanze di Dispatcher. Le configurazioni con più istanze di Dispatcher devono essere considerate attentamente, perché un’ulteriore istanza di Dispatcher può aumentare il carico sulle istanze di pubblicazione disponibili e può facilmente diminuire le prestazioni nella maggior parte delle applicazioni.

## Bilanciamento del carico in Dispatcher {#how-the-dispatcher-performs-load-balancing}

### Statistiche sulle prestazioni

Dispatcher mantiene statistiche interne sulla velocità di elaborazione dei documenti in ogni istanza di AEM. In base a questi dati, Dispatcher valuta quale istanza può fornire il tempo di risposta più rapido quando risponde a una richiesta e quindi riserva il tempo di calcolo necessario su tale istanza.

Diversi tipi di richieste possono presentare tempi medi di completamento diversi, pertanto Dispatcher consente di specificare le categorie di documenti. Queste categorie vengono quindi considerate quando si calcolano le stime del tempo. Ad esempio, è possibile distinguere tra pagine e immagini di HTML, in quanto i tempi di risposta tipici potrebbero essere molto diversi.

Se utilizzi una funzione di ricerca elaborata, puoi creare una categoria per le query di ricerca. Questo metodo consente al Dispatcher di inviare le query di ricerca all’istanza che risponde più velocemente. Inoltre, aiuta a evitare che un’istanza più lenta si fermi quando riceve diverse query di ricerca &quot;costose&quot;, mentre le altre ottengono le richieste &quot;più economiche&quot;.

### Contenuto personalizzato (connessioni permanenti)

Con le connessioni permanenti è possibile garantire che i documenti di un utente siano tutti composti nella stessa istanza di AEM. Questo punto è importante se utilizzi pagine personalizzate e dati di sessione. I dati vengono archiviati nell’istanza, per cui le richieste successive dello stesso utente devono tornare a quell’istanza, altrimenti i dati andranno persi.

Poiché con le connessioni permanenti la capacità di Dispatcher di ottimizzare le richieste risulta limitata, è consigliabile utilizzarle solo quando necessario. Puoi specificare la cartella che contiene i documenti “permanenti”, in modo da garantire che tutti i documenti in tale cartella vengano composti nella stessa istanza per ogni utente.

>[!NOTE]
>
>Per la maggior parte delle pagine che utilizzano connessioni permanenti è necessario disattivare la cache, altrimenti la pagina risulterà uguale per tutti gli utenti, indipendentemente dal contenuto della sessione.
>
>Per *alcune* applicazioni, può essere possibile utilizzare sia connessioni permanenti che il caching, ad esempio, se si visualizza un modulo che scrive i dati nella sessione.

## Utilizzo di più istanze di Dispatcher {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più istanze di Dispatcher. Ad esempio, puoi utilizzare:

* Un’istanza di Dispatcher per pubblicare un sito web nella Intranet.
* Una seconda istanza di Dispatcher, con indirizzo e impostazioni di sicurezza diversi, per pubblicare lo stesso contenuto in Internet.

In questo caso, accertati che ogni richiesta venga gestita tramite un’unica istanza di Dispatcher. Un’istanza di Dispatcher non gestisce le richieste provenienti da un’altra istanza di Dispatcher. Accertati pertanto che entrambe le istanze di Dispatcher accedano direttamente al sito web di AEM.

## Utilizzo di Dispatcher con una rete CDN {#using-dispatcher-with-a-cdn}

Una rete CDN (Content Delivery Network), come Akamai Edge Delivery o Amazon Cloud Front, consente di distribuire contenuto da una località vicina all’utente finale. Ciò permette di:

* Velocizzare i tempi di risposta per gli utenti finali.
* Alleggerisce il carico dei server.

Come componente di infrastruttura HTTP, una rete CDN funziona in modo analogo a Dispatcher. Quando un nodo CDN riceve una richiesta, invia la richiesta dalla propria cache, se possibile (la risorsa è disponibile nella cache ed è valida). In caso contrario, si rivolge al server successivo più vicino per recuperare la risorsa e memorizzarla in cache per eventuali ulteriori richieste.

Il “server successivo più vicino” dipende dalla configurazione specifica. Ad esempio, in una configurazione di Akamai il percorso della richiesta può essere il seguente:

* Nodo Akamai Edge
* Livello Akamai Midgres
* Firewall dell’utente
* Load balancer dell’utente
* Dispatcher
* AEM

Di solito, Dispatcher è il server successivo che potrebbe elaborare il documento da una cache e influenzare le intestazioni di risposta restituite al server CDN.

## Controllo di una cache della rete CDN {#controlling-a-cdn-cache}

Esistono diversi modi per controllare per quanto tempo una rete CDN memorizza in cache una risorsa prima che venga recuperata da Dispatcher.

1. Configurazione esplicita\
   Configura per quanto tempo determinate risorse vengono mantenute nella cache della rete CDN, a seconda del tipo di MIME, dell’estensione, del tipo di richiesta e così via.

1. Intestazioni di scadenza e controllo della cache\
   L&#39;onore della maggior parte delle CDN `Expires:` e `Cache-Control:` Intestazioni HTTP se inviate dal server upstream. Questo metodo può essere ottenuto, ad esempio, utilizzando [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) Modulo Apache.

1. Annullamento manuale della validità\
   Le reti CDN consentono di rimuovere risorse dalla cache tramite interfacce web.
1. Annullamento della validità basato su API\
   La maggior parte delle reti CDN offre anche un’API REST e/o SOAP che consente di rimuovere le risorse dalla cache.

In una tipica configurazione di AEM, la configurazione per estensione, per percorso o per entrambi - ottenibile tramite i punti 1 e 2 di cui sopra - offre la possibilità di impostare periodi di caching ragionevoli per le risorse utilizzate spesso che non cambiano spesso, come immagini di progettazione e librerie client. Quando vengono distribuite nuove versioni, in genere è necessario un annullamento manuale della validità.

Se si utilizza questo approccio per memorizzare in cache il contenuto gestito, le modifiche apportate al contenuto sono visibili solo agli utenti finali dopo la scadenza del periodo di caching configurato e il documento viene di nuovo recuperato da Dispatcher.

Per un controllo più granulare, l’annullamento della validità basato su API consente di annullare la validità della cache di una rete CDN quando la cache del Dispatcher viene invalidata. In base all’API della rete CDN, puoi implementare la tua [ContentBuilder](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/replication/ContentBuilder.html) e [TransportHandler](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/replication/TransportHandler.html) (se l’API non è basata su REST) e imposta un agente di replica che utilizza questi elementi per annullare la validità della cache della rete CDN.

>[!NOTE]
>
>Vedi anche la presentazione su [Sicurezza di AEM (CQ) Dispatcher e caching di rete CDN e browser](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015) nonché la presentazione registrata sul [caching in Dispatcher](https://experienceleague.adobe.com/docs/experience-manager-gems-events/gems/gems2015/aem-dispatcher-caching-new-features-and-optimizations.html?lang=en).

## Utilizzo di un’istanza di Dispatcher con Author Server {#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>Se utilizzi [AEM con interfaccia touch](https://experienceleague.adobe.com/docs/experience-manager-65/developing/introduction/touch-ui-concepts.html?lang=en), do **not** contenuto dell’istanza dell’autore della cache. Se la memorizzazione in cache è stata abilitata per l’istanza di authoring, devi disattivarla ed eliminare il contenuto della directory cache. Per disattivare la memorizzazione in cache, modifica la `author_dispatcher.any` e modificare il `/rule` proprietà `/cache` come segue:

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

Puoi utilizzare un’istanza di Dispatcher prima di un’istanza Autore per migliorare le prestazioni di authoring. Per configurare un’istanza di Dispatcher per l’authoring, effettua le seguenti operazioni:

1. Installa un Dispatcher in un server web (un server web Apache o IIS, vedi [Installazione di Dispatcher](dispatcher-install.md)).
1. Testa il Dispatcher appena installato rispetto a un&#39;istanza di pubblicazione AEM funzionante. In questo modo si garantisce che sia stata effettuata un&#39;installazione corretta della linea di base.
1. A questo punto accertati che Dispatcher possa connettersi all’istanza Autore tramite TCP/IP.
1. Sostituire l&#39;esempio `dispatcher.any` con il `author_dispatcher.any` file fornito con [Download di Dispatcher](release-notes.md#downloads).
1. Apri il file `author_dispatcher.any` in un editor di testo e apporta le seguenti modifiche:

   1. Modificare la `/hostname` e `/port` del `/renders` in modo che puntino all’istanza di authoring.
   1. Modificare la `/docroot` del `/cache` in modo che puntino a una directory cache. Se utilizzi [AEM con l’interfaccia utente touch](https://experienceleague.adobe.com/docs/experience-manager-65/developing/introduction/touch-ui-concepts.html?lang=en), vedi l’avviso riportato sopra.
   1. Salva le modifiche.

1. Elimina tutti i file esistenti nella directory `/cache` > `/docroot` configurata in precedenza.
1. Riavvia il server web.

>[!NOTE]
>
>Con le `author_dispatcher.any` configurazione, quando installi un pacchetto di funzioni, un hotfix o un pacchetto di codice dell&#39;applicazione CQ5 che interessa qualsiasi contenuto in `/libs` o `/apps`, devi eliminare i file memorizzati nella cache sotto tali directory nella cache del Dispatcher. In questo modo i file appena aggiornati vengono recuperati e non i file memorizzati nella cache precedenti alla successiva richiesta.

>[!CAUTION]
>
>Se hai utilizzato Dispatcher per l’authoring configurato in precedenza e hai abilitato un *Agente di svuotamento del dispatcher*, procedi come segue:

1. Elimina o disattiva la **autore Dispatcher** flush dell&#39;agente sull&#39;istanza di authoring AEM.
1. Ripeti la configurazione dell’istanza di Dispatcher per l’authoring seguendo le nuove istruzioni riportate sopra.

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
