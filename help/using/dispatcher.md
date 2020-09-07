---
title: Panoramica di Dispatcher
seo-title: Panoramica di Adobe AEM Dispatcher
description: Questo articolo fornisce una panoramica generale di Dispatcher.
seo-description: Questo articolo fornisce una panoramica generale di Adobe Experience Manager Dispatcher.
uuid: 71766f86-5e91-446b-a078-061b179d090d
pageversionid: 1193211344162
topic-tags: dispatcher
content-type: reference
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
translation-type: tm+mt
source-git-commit: 590a9a2b6005cd48676cf525b0e029e42d1b3ca6
workflow-type: tm+mt
source-wordcount: '3199'
ht-degree: 90%

---


# Panoramica di Dispatcher {#dispatcher-overview}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

Dispatcher è uno strumento di cache e/o bilanciamento del carico Adobe Experience Manager che può essere utilizzato insieme a un server Web di classe enterprise.

Il processo di implementazione del dispatcher è indipendente dal server Web e dalla piattaforma del sistema operativo prescelti:

1. Scopri di più su Dispatcher (questa pagina). Inoltre, vedete le domande [frequenti sul dispatcher](https://helpx.adobe.com/experience-manager/using/dispatcher-faq.html).
1. Installate un server [Web](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html) supportato in base alla documentazione del server Web.
1. [Installa il modulo Dispatcher](dispatcher-install.md) nel server web e configura di conseguenza il server web.
1. [Configura Dispatcher](dispatcher-configuration.md) (il file dispatcher.any).
1. [Configura AEM](page-invalidate.md) in modo che gli aggiornamenti dei contenuti annullino la validità della cache.

>[!NOTE]
>
>Per comprendere meglio come funziona Dispatcher con AEM:
>
>* Consulta [Ask the AEM Community Experts for luglio 2017](https://bit.ly/ATACE0717).
>* Accedere [a questo archivio](https://github.com/adobe/aem-dispatcher-experiments). Contiene una raccolta di esperimenti in un formato di laboratorio &quot;da portare a casa&quot;.



Utilizza le seguenti informazioni come richiesto:

* [Elenco di controllo della sicurezza di Dispatcher](security-checklist.md)
* [La Knowledge Base del dispatcher](https://helpx.adobe.com/cq/kb/index/dispatcher.html)
* [Ottimizzazione di un sito Web per le prestazioni della cache](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html)
* [Utilizzo di Dispatcher con più domini](dispatcher-domains.md)
* [Utilizzo di SSL con Dispatcher](dispatcher-ssl.md)
* [Implementazione del caching soggetto alle autorizzazioni](permissions-cache.md)
* [Risoluzione dei problemi di Dispatcher](dispatcher-troubleshooting.md)
* [Domande frequenti sui principali problemi di Dispatcher](dispatcher-faq.md)

>[!NOTE]
>
>**Dispatcher viene utilizzato in genere** per memorizzare in cache le risposte di un’**istanza di pubblicazione** di AEM, per aumentare la capacità di risposta e la sicurezza del sito web pubblicato rivolto verso l’esterno. La maggior parte della discussione è incentrata su questo caso specifico.
>
>È però anche possibile utilizzare Dispatcher per incrementare la reattività dell’**istanza di authoring**, in particolare se il sito web viene modificato e aggiornato da un numero elevato di utenti. Per i dettagli specifici di questo caso, vedere [Utilizzo di Dispatcher con Author Server](#using-a-dispatcher-with-an-author-server), di seguito.

## Perché usare Dispatcher per implementare il caching? {#why-use-dispatcher-to-implement-caching}

Sono due gli approcci di base alla pubblicazione sul web:

* **Server web statici: come Apache o IIS, che sono molto semplici, ma veloci.**
* **Server di gestione dei contenuti**: che forniscono contenuti intelligenti, dinamici e in tempo reale, ma richiedono un tempo di calcolo maggiore e altre risorse.

Dispatcher permette di realizzare un ambiente al tempo stesso veloce e dinamico. Viene utilizzato in parte come un server HTML statico, ad esempio Apache, con lo scopo di:

* archiviare (o “memorizzare in cache”) la maggior parte dei contenuti del sito sotto forma di sito web statico;
* accedere il meno possibile al motore di layout.

Questo significa che:

* il **contenuto statico** viene gestito in modo semplice e rapido analogamente a un server web statico;*è inoltre possibile utilizzare gli strumenti di amministrazione e sicurezza disponibili per i server web statici*;

* il **contenuto dinamico** viene generato in base alle necessità, senza rallentare il sistema più del necessario.

Dispatcher contiene meccanismi per generare e aggiornare il codice HTML statico in base al contenuto del sito dinamico. È possibile specificare in dettaglio quali documenti vengono archiviati come file statici e quali vengono sempre generati in modo dinamico.

Questa sezione illustra i principi alla base di tale funzionamento.

### Server web statico {#static-web-server}

![](assets/chlimage_1-3.png)

Un server web statico, come Apache o IIS, viene utilizzato per distribuire file HTML statici ai visitatori del sito web. Le pagine statiche vengono create una sola volta, quindi a ogni richiesta verrà distribuito lo stesso contenuto.

Questo processo è molto semplice e quindi estremamente efficiente. Se un visitatore richiede un file, ad esempio una pagina HTML, in genere il file viene prelevato direttamente dalla memoria o nel peggiore dei casi viene letto dall’unità locale. I server web statici sono disponibili da molto tempo, per cui esistono già numerosi strumenti per la gestione dell’amministrazione e della sicurezza; sono inoltre perfettamente integrati con le infrastrutture di rete.

### Server di gestione dei contenuti {#content-management-servers}

![](assets/chlimage_1-4.png)

Se utilizzi un server di gestione dei contenuti, come AEM, la richiesta di un visitatore viene elaborata da un motore di layout avanzato. Il motore legge il contenuto da un archivio che, insieme a stili, formati e diritti di accesso, trasforma il contenuto in un documento personalizzato in base alle esigenze e ai diritti del visitatore.

Questo ti permette di creare contenuti più ricchi e dinamici, che aumentano la flessibilità e la funzionalità del tuo sito web. Tuttavia, il motore di layout richiede una potenza di elaborazione maggiore rispetto a un server statico, pertanto questa configurazione potrebbe essere soggetta a rallentamenti se molti visitatori utilizzano il sistema.

## Funzionamento del caching in Dispatcher{#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**Directory cache** Per il caching il modulo Dispatcher utilizza la funzionalità del server web che consente di distribuire contenuto statico. Dispatcher inserisce i documenti memorizzati in cache nella directory principale dei documenti del server web.

>[!NOTE]
>
>Se manca la configurazione per il caching delle intestazioni HTTP, Dispatcher archivia solo il codice HTML della pagina e non le intestazioni HTTP. Questo comportamento può costituire un problema se utilizzi codifiche diverse all’interno del sito web, in quanto tali codifiche potrebbero andare perse. Per abilitare il caching delle intestazioni HTTP, consulta [Configurazione della cache di Dispatcher.](https://helpx.adobe.com/it/experience-manager/dispatcher/using/dispatcher-configuration.html)

>[!NOTE]
>
>Se la directory principale dei documenti del server web si trova in un server di accesso alla rete (NAS, Network Attached Storage), potresti riscontrare un degrado delle prestazioni. Quando inoltre una directory principale dei documenti che si trova nel NAS viene condivisa tra più server web, possono verificarsi blocchi intermittenti durante l’esecuzione di azioni di replica.

>[!NOTE]
>
>Dispatcher archivia il documento memorizzato in cache in una struttura uguale all’URL richiesto.
>
>Il sistema operativo può prevedere limitazioni per la lunghezza del nome file, ad esempio se un URL contiene molti selettori.

### Metodi di caching

In Dispatcher sono disponibili due metodi principali per aggiornare il contenuto della cache quando vengono apportate modifiche al sito web.

* **Aggiornamenti del contenuto**: le pagine modificate e i file che sono direttamente associati ad esse vengono rimossi.
* **Annullamento automatico della validità**: le parti della cache che potrebbero risultare obsolete dopo un aggiornamento vengono invalidate automaticamente, ovvero le pagine pertinenti vengono contrassegnate come non aggiornate ma non viene eseguita alcuna operazione di eliminazione.

### Aggiornamenti del contenuto

In un aggiornamento del contenuto uno o più documenti AEM cambiano. AEM invia una richiesta di sindacazione a Dispatcher, che aggiorna la cache di conseguenza:

1. Elimina i file modificati dalla cache.
1. Elimina dalla cache tutti i file che iniziano con lo stesso handle. Ad esempio, se viene aggiornato il file /en/index.html, vengono eliminati tutti i file che iniziano con “/en/index.” . Questo meccanismo consente di progettare siti efficienti dal punto di vista della cache, soprattutto per quanto riguarda la navigazione nelle immagini.
1. Il cosiddetto **statfile** viene *toccato* e la sua marca temporale viene aggiornata per indicare la data dell’ultima modifica.

Tieni presente le seguenti considerazioni:

* Gli aggiornamenti di contenuto vengono in genere utilizzati insieme a un sistema di authoring che “sa” cosa deve essere sostituito.
* I file interessati da un aggiornamento del contenuto vengono rimossi, ma non sostituiti immediatamente. La prossima volta che un tale file viene richiesto, Dispatcher recupera il nuovo file dall’istanza di AEM e lo inserisce nella cache, sovrascrivendo il vecchio contenuto.
* In genere, le immagini generate automaticamente che incorporano testo di una pagina vengono archiviate in file di immagine che iniziano con lo stesso handle, in modo da garantire l’associazione per l’eliminazione. Ad esempio, puoi archiviare il testo del titolo della pagina mypage.html come immagine mypage.titlePicture.gif nella stessa cartella. In questo modo l’immagine viene eliminata automaticamente dalla cache ogni volta che la pagina viene aggiornata e potrai essere sicuro che l’immagine rispecchi sempre la versione corrente della pagina.
* Possono esistere diversi statfile, ad esempio uno per cartella di lingua. Se una pagina viene aggiornata, AEM cerca la cartella padre successiva contenente uno statfile e *tocca* tale file.

### Annullamento automatico della validità

Con l’annullamento automatico della validità, parti della cache vengono automaticamente invalidate, ma non viene fisicamente eliminato alcun file. Ad ogni aggiornamento del contenuto, il cosiddetto statfile viene toccato, e la marca temporale corrispondente rispecchia quindi l’ultimo aggiornamento del contenuto.

In Dispatcher è disponibile un elenco di file soggetti ad annullamento automatico della validità. Quando viene richiesto un documento da tale elenco, Dispatcher confronta la data del documento memorizzato in cache con la marca temporale dello statfile:

* se il documento memorizzato in cache è più recente, Dispatcher lo restituisce;
* se è precedente, Dispatcher recupera la versione corrente dall’istanza di AEM.

Anche in questo caso, tieni presente le seguenti considerazioni:

* L’annullamento automatico della validità viene utilizzato in genere quando le interrelazioni sono complesse, ad esempio per le pagine HTML. Queste pagine contengono collegamenti e voci di navigazione e in genere devono essere aggiornate dopo un aggiornamento del contenuto. Se sono presenti file PDF o di immagine generati automaticamente, puoi scegliere di annullare automaticamente la validità anche di tali file.
* L’annullamento automatico della validità non richiede alcuna azione da parte di Dispatcher al momento dell’aggiornamento, ad eccezione della modifica tramite touch dello statfile. Tuttavia, quando lo statfile viene toccato, il contenuto della cache diventa automaticamente obsoleto, anche se non viene rimosso fisicamente dalla cache.

## Restituzione dei documenti in Dispatcher {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### Determinare se un documento è soggetto a caching

È possibile [definire i documenti memorizzati nella cache del Dispatcher nel file](https://helpx.adobe.com/it/experience-manager/dispatcher/using/dispatcher-configuration.html)di configurazione. Dispatcher confronta la richiesta con l’elenco dei documenti memorizzabili in cache. Se il documento non è incluso in questo elenco, Dispatcher richiede il documento dall’istanza di AEM.

Dispatcher richiede *sempre* il documento direttamente all’istanza di AEM nei casi seguenti:

* Se l’URI della richiesta contiene il punto interrogativo “?”. In genere questo segno indica una pagina dinamica, ad esempio un risultato della ricerca, che non deve essere memorizzata in cache.
* Se manca l’estensione del file. Il server web ha bisogno dell’estensione per determinare il tipo di documento (tipo MIME).
* Se l’intestazione di autenticazione è impostata (configurabile).

>[!NOTE]
>
>Dispatcher può memorizzare in cache i metodi GET o HEAD (per l’intestazione HTTP). Per ulteriori informazioni sul caching delle intestazioni delle risposte, consultate la sezione [Memorizzazione in cache delle intestazioni](https://helpx.adobe.com/it/experience-manager/dispatcher/using/dispatcher-configuration.html) di risposta HTTP.

### Determinare se un documento è memorizzato in cache

Dispatcher archivia nel server web i file memorizzati in cache come se facessero parte di un sito web statico. Se un utente richiede un documento memorizzabile in cache, Dispatcher controlla se tale documento è presente nel file system del server web:

* Se il documento è memorizzato in cache, Dispatcher restituisce il file.
* Se non è memorizzato in cache, Dispatcher richiede il documento dall’istanza di AEM.

### Determinare se un documento è aggiornato

Per verificare se un documento è aggiornato, Dispatcher esegue due passaggi:

1. Verifica se il documento è soggetto ad annullamento automatico della validità. Se non lo è, il documento viene considerato aggiornato.
1. Se il documento è configurato per l’annullamento automatico della validità, Dispatcher controlla se è più o meno recente dell’ultima modifica disponibile. Se è meno recente, Dispatcher richiede la versione corrente dall’istanza di AEM e sostituisce la versione nella cache.

>[!NOTE]
>
>I documenti senza **annullamento automatico della validità** rimangono nella cache fino a quando non vengono fisicamente eliminati, ad esempio tramite un aggiornamento del contenuto del sito web.

## Vantaggi del bilanciamento del carico {#the-benefits-of-load-balancing}

Per bilanciamento del carico si intende la distribuzione del carico di elaborazione del sito web tra più istanze di AEM.

![](assets/chlimage_1-7.png)

I vantaggi di questa pratica sono i seguenti:

* **Maggiore potenza di elaborazione**
In pratica Dispatcher condivide le richieste di documenti tra più istanze di AEM. Poiché il numero di documenti da elaborare per ogni istanza è ridotto, i tempi di risposta risultano inferiori. Dispatcher mantiene statistiche interne per ogni categoria di documenti, in modo da poter stimare il carico e distribuire le query in modo efficiente.

* **Maggiore copertura in caso di errore**
Se non riceve risposte da un’istanza, Dispatcher inoltrerà automaticamente le richieste a una delle altre istanze. Pertanto, se un’istanza risulta non disponibile, l’unico effetto è un rallentamento del sito, proporzionato alla potenza di elaborazione persa, ma l’elaborazione di tutti i servizi non subirà interruzioni.

* è anche possibile gestire siti web diversi nello stesso server web statico.

>[!NOTE]
>
>Mentre il bilanciamento del carico consente di distribuire il carico in modo efficiente, la memorizzazione in cache contribuisce a ridurlo. Prova quindi a ottimizzare il caching e ridurre il carico complessivo prima di configurare il bilanciamento del carico. Un caching appropriato può contribuire a migliorare le prestazioni del load balancer o rendere superfluo il bilanciamento del carico.

>[!CAUTION]
>
>Anche se un’unica istanza di Dispatcher è in genere in grado di assorbire la capacità delle istanze di pubblicazione disponibili, per alcune rare applicazioni può essere utile bilanciare ulteriormente il carico tra due istanze di Dispatcher. È necessario prestare particolare attenzione alle configurazioni con più istanze di Dispatcher, poiché un’ulteriore istanza di Dispatcher implica un aumento del carico sulle istanze di pubblicazione disponibili e può facilmente causare un calo delle prestazioni nella maggior parte delle applicazioni.

## Bilanciamento del carico in Dispatcher {#how-the-dispatcher-performs-load-balancing}

### Statistiche sulle prestazioni

Dispatcher mantiene statistiche interne sulla velocità di elaborazione dei documenti in ogni istanza di AEM. In base a questi dati, Dispatcher può stimare quale istanza è in grado di rispondere più rapidamente a una richiesta e quindi riserva il tempo di elaborazione necessario a tale istanza.

I tempi medi di completamento dei diversi tipi di richieste possono essere diversi, pertanto Dispatcher consente di specificare le categorie dei documenti. Queste vengono quindi considerate per calcolare i tempi stimati. Ad esempio, puoi fare una distinzione tra pagine HTML e immagini, poiché i tempi di risposta tipici potrebbero essere molto diversi.

Se utilizzi una funzione di ricerca elaborata, puoi creare una nuova categoria per le query di ricerca. In questo modo Dispatcher può inviare le query di ricerca all’istanza che risponde più velocemente e si impedisce a un’istanza più lenta di rimanere in sospeso quando riceve diverse query di ricerca “onerose”, mentre altre istanze ricevono altre richieste “meno onerose”.

### Contenuto personalizzato (connessioni permanenti)

Con le connessioni permanenti è possibile garantire che i documenti di un utente siano tutti composti nella stessa istanza di AEM. Questa caratteristica è importante se utilizzi pagine personalizzate e dati di sessione. I dati vengono archiviati nell’istanza, per cui le richieste successive dello stesso utente devono tornare all’istanza; in caso contrario, i dati andranno persi.

Poiché con le connessioni permanenti la capacità di Dispatcher di ottimizzare le richieste risulta limitata, è consigliabile utilizzarle solo quando necessario. Puoi specificare la cartella che contiene i documenti “permanenti”, in modo da garantire che tutti i documenti in tale cartella vengano composti nella stessa istanza per ogni utente.

>[!NOTE]
>
>Per la maggior parte delle pagine che utilizzano connessioni permanenti è necessario disattivare la cache, altrimenti la pagina risulterà uguale per tutti gli utenti, indipendentemente dal contenuto della sessione.
>
>Per *alcune* applicazioni, può essere possibile utilizzare sia connessioni permanenti che il caching, ad esempio, se si visualizza un modulo che scrive i dati nella sessione.

## Utilizzo di più istanze di Dispatcher {#using-multiple-dispatchers}

In configurazioni complesse è possibile utilizzare più istanze di Dispatcher. Ad esempio, puoi utilizzare:

* un’istanza di Dispatcher per pubblicare un sito web nella Intranet;
* una seconda istanza di Dispatcher, con un indirizzo diverso e impostazioni di sicurezza diverse, per pubblicare lo stesso contenuto in Internet.

In tal caso, assicurati che ogni richiesta venga gestita tramite un’unica istanza di Dispatcher. Un’istanza di Dispatcher non gestisce le richieste provenienti da un’altra istanza di Dispatcher. Assicurati pertanto che entrambe le istanze di Dispatcher accedano direttamente al sito web AEM.

## Utilizzo di Dispatcher con una rete CDN {#using-dispatcher-with-a-cdn}

Una rete CDN (Content Delivery Network), come Akamai Edge Delivery o Amazon Cloud Front, consente di distribuire contenuto da una posizione vicina all’utente finale. Per questo motivo:

* velocizza i tempi di risposta per gli utenti finali;
* alleggerisce il carico dei server.

In quanto componente dell’infrastruttura HTTP, una rete CDN funziona in modo analogo a Dispatcher: quando un nodo della rete CDN riceve una richiesta, la distribuisce dalla cache se possibile, ovvero se la risorsa è disponibile nella cache ed è valida. In caso contrario, si rivolge al server successivo più vicino per recuperare la risorsa e memorizzarla in cache per ulteriori richieste, a seconda dei casi.

Il “server successivo più vicino” dipende dalla configurazione specifica. Ad esempio, in una configurazione di Akamai il percorso della richiesta può essere il seguente:

* Nodo Akamai Edge
* Livello Akamai Midgres
* Firewall dell’utente
* Load balancer dell’utente
* Dispatcher
* AEM

Nella maggior parte dei casi, Dispatcher è il server successivo che può distribuire il documento da una cache e influire sulle intestazioni di risposta restituite al server della rete CDN.

## Controllo di una cache della rete CDN {#controlling-a-cdn-cache}

Sono disponibili diversi modi per controllare per quanto tempo una rete CDN memorizzerà in cache una risorsa prima che venga recuperata da Dispatcher.

1. Configurazione esplicita\
   Configura per quanto tempo determinate risorse vengono mantenute nella cache della rete CDN, a seconda del tipo di MIME, dell’estensione, del tipo di richiesta e così via.

1. Intestazioni di scadenza e controllo della cache\
   La maggior parte delle reti CDN rispetta le intestazioni HTTP `Expires:` e `Cache-Control:` se inviate dal server upstream. This can be achieved e.g. by using the [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) Apache Module.

1. Annullamento manuale della validità\
   Le reti CDN consentono di rimuovere risorse dalla cache tramite interfacce web.
1. Annullamento della validità basato su API\
   La maggior parte delle reti CDN offre anche un’API REST e/o SOAP che consente di rimuovere le risorse dalla cache.

In una tipica configurazione di AEM, la configurazione per estensione e/o per percorso, ottenibile tramite i punti 1 e 2 di cui sopra, offre la possibilità di impostare periodi di caching ragionevoli per risorse utilizzate spesso non soggette a modifiche frequenti, come immagini di progettazione e librerie client. Quando vengono distribuite nuove versioni, in genere è necessario un annullamento manuale della validità.

Se si utilizza questo approccio per memorizzare in cache il contenuto gestito, le modifiche apportate al contenuto sono visibili solo agli utenti finali dopo la scadenza del periodo di caching configurato e il documento viene recuperato da Dispatcher.

Per un controllo più granulare, l’annullamento della validità basato su API consente di annullare la validità della cache di una rete CDN appena viene annullata la validità della cache di Dispatcher. Based on the CDNs API, you can implement your own [ContentBuilder](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/day/cq/replication/ContentBuilder.html) and [TransportHandler](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/day/cq/replication/TransportHandler.html) (if the API is not REST-based) and set up a Replication Agent that will use these to invalidate the CDN&#39;s cache.

>[!NOTE]
>
>See also [AEM (CQ) Dispatcher Security and CDN+Browser Caching](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015) and recorded presentation on [Dispatcher Caching](https://docs.adobe.com/content/ddc/en/gems/dispatcher-caching---new-features-and-optimizations.html).

## Utilizzo di un’istanza di Dispatcher con Author Server{#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>if you are using [AEM with Touch UI](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html) you should **not** cache author instance content. Se è stato abilitato il caching per l’istanza di authoring, devi disattivarlo ed eliminare il contenuto della directory cache. Per disattivare il caching, devi modificare il file `author_dispatcher.any` e modificare la proprietà `/rule` della sezione `/cache` nel modo seguente:

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

Puoi utilizzare un’istanza di Dispatcher prima di un’istanza di authoring per migliorare le prestazioni di authoring. Per configurare un’istanza di Dispatcher per l’authoring, effettua le seguenti operazioni:

1. Installa un’istanza di Dispatcher in un server web (può essere un server web Apache o IIS; consulta [Installazione di Dispatcher](dispatcher-install.md)).
1. Puoi testare l’istanza di Dispatcher appena installata in base a un’istanza di pubblicazione funzionante di AEM, per verificare che sia stata raggiunta un’installazione corretta della linea di base.
1. A questo punto assicurati che Dispatcher sia in grado di connettersi all’istanza di authoring tramite TCP/IP.
1. Sostituisci il file dispatcher.any di esempio con il file author_dispatcher.any fornito con il [download di Dispatcher](release-notes.md#downloads).
1. Apri il file `author_dispatcher.any` in un editor di testo e apporta le seguenti modifiche:

   1. Modifica le istruzioni `/hostname` e `/port` della sezione `/renders` in modo che puntino all’istanza di authoring.
   1. Modifica l’istruzione `/docroot` della sezione `/cache` in modo che punti a una directory cache. Se utilizzi [AEM con l’interfaccia](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html)touch, vedi l’avviso qui sopra.
   1. Salva le modifiche.

1. Elimina tutti i file esistenti nella directory `/cache` > `/docroot` configurata in precedenza.
1. Riavvia il server web.

>[!NOTE]
>
>Tieni presente che con la configurazione fornita di `author_dispatcher.any`, quando installi un pacchetto di funzioni, un aggiornamento rapido o un pacchetto di codice dell’applicazione CQ5 che interessa contenuto presente in `/libs` o `/apps`, devi eliminare i file memorizzati in tali directory nella cache di Dispatcher in modo che, la prossima volta che vengono richiesti, vengano recuperati i file aggiornati e non quelli memorizzati in cache.

>[!CAUTION]
>
>Se hai usato un’istanza di Dispatcher per l’authoring configurata in precedenza e hai attivato un *agente di eliminazione Dispatcher*, effettua le seguenti operazioni:

1. Elimina o disattiva l’agente di eliminazione dell’**istanza di Dispatcher** per l’authoring nell’istanza di authoring di AEM.
1. Ripeti la configurazione dell’istanza di Dispatcher per l’authoring seguendo le nuove istruzioni riportate sopra.

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
