---
title: Panoramica del dispatcher
seo-title: Panoramica del dispatcher Adobe AEM
description: Questo articolo fornisce una panoramica generale del dispatcher.
seo-description: Questo articolo fornisce una panoramica generale del dispatcher Adobe Experience Manager.
uuid: 71766f86-5e91-446b-a078-061b179d090d
pageversionid: '1193211344162'
topic-tags: spedizioniere
content-type: riferimento
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
translation-type: tm+mt
source-git-commit: de6a513baf3e6b1a1463a442fa840e59f2196e8e

---


# Panoramica del dispatcher {#dispatcher-overview}

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Potreste essere stati reindirizzati a questa pagina se avete seguito un collegamento alla documentazione del dispatcher incorporata nella documentazione di una versione precedente di AEM.

Dispatcher è lo strumento di cache e/o bilanciamento del carico di Adobe Experience Manager. L’utilizzo del dispatcher di AEM consente inoltre di proteggere il server AEM dagli attacchi. Di conseguenza, puoi aumentare la sicurezza dell’istanza AEM utilizzando il dispatcher insieme a un server Web di classe enterprise.

Il processo di implementazione di un dispatcher è indipendente dal server Web e dalla piattaforma del sistema operativo prescelti:

1. Ulteriori informazioni sul dispatcher (questa pagina). Inoltre, vedete le domande [frequenti sul dispatcher](https://helpx.adobe.com/experience-manager/using/dispatcher-faq.html).
1. Installate un server [Web](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html) supportato in base alla documentazione del server Web.

1. [Installate il modulo](dispatcher-install.md) Dispatcher sul server Web e configurate di conseguenza il server Web.
1. [Configurare il dispatcher](dispatcher-configuration.md) (il file dispatcher.any).

1. [Configurate AEM](page-invalidate.md) in modo che gli aggiornamenti dei contenuti invalidino la cache.

>[!NOTE]
>
>Per capire meglio come funziona il dispatcher con AEM, consulta [Chiedere agli esperti della community AEM per luglio 2017](https://bit.ly/ATACE0717).

Utilizzate le seguenti informazioni come richiesto:

* [Elenco Di Controllo Del Dispatcher](security-checklist.md)
* [La Knowledge Base del dispatcher](https://helpx.adobe.com/cq/kb/index/dispatcher.html)
* [Ottimizzazione di un sito Web per le prestazioni della cache](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html)
* [Utilizzo del dispatcher con più domini](dispatcher-domains.md)
* [Utilizzo di SSL con Dispatcher](dispatcher-ssl.md)
* [Implementazione della memorizzazione delle autorizzazioni nella cache](permissions-cache.md)
* [Risoluzione dei problemi del dispatcher](dispatcher-troubleshooting.md)
* [Domande frequenti sui problemi principali del dispatcher](dispatcher-faq.md)

>[!NOTE]
>
>**L’utilizzo più comune di Dispatcher** consiste nel memorizzare nella cache le risposte da un’istanza **di** pubblicazione AEM, per aumentare la capacità di risposta e la sicurezza del sito Web pubblicato rivolto esternamente. La maggior parte della discussione si concentra su questo caso.
>
>Tuttavia, il Dispatcher può essere utilizzato anche per aumentare la reattività dell’istanza **di** authoring, in particolare se si dispone di un numero elevato di utenti che modificano e aggiornano il sito Web. Per informazioni dettagliate su questo caso, consultate [Utilizzo di un dispatcher con un server](#using-a-dispatcher-with-an-author-server)Author, di seguito.

## Perché utilizzare Dispatcher per implementare la cache? {#why-use-dispatcher-to-implement-caching}

Esistono due approcci di base alla pubblicazione Web:

* **Server** Web statici: come Apache o IIS, sono molto semplici, ma veloci.
* **Server** di gestione dei contenuti: che forniscono contenuti dinamici, in tempo reale e intelligenti, ma richiedono molto più tempo di calcolo e altre risorse.

Il dispatcher consente di realizzare un ambiente sia veloce che dinamico. Funziona come parte di un server HTML statico, come Apache, allo scopo di:

* la memorizzazione (o "memorizzazione nella cache") del contenuto del sito come possibile, sotto forma di sito Web statico
* accedere al modulo di gestione del layout il meno possibile.

Ciò significa che:

* **il contenuto** statico viene gestito con la stessa velocità e facilità di un server Web statico;*inoltre, potete utilizzare gli strumenti di amministrazione e protezione disponibili per i server Web statici*.

* **il contenuto** dinamico viene generato in base alle esigenze, senza rallentare il sistema più del necessario.

Il Dispatcher contiene meccanismi per generare e aggiornare codice HTML statico basato sul contenuto del sito dinamico. Potete specificare in dettaglio quali documenti vengono memorizzati come file statici e quali vengono sempre generati in modo dinamico.

Questa sezione illustra i principi alla base di ciò.

### Server Web statico {#static-web-server}

![](assets/chlimage_1-3.png)

Un server Web statico, come Apache o IIS, distribuisce ai visitatori del sito Web file HTML statici. Le pagine statiche vengono create una volta, quindi lo stesso contenuto verrà distribuito per ogni richiesta.

Questo processo è molto semplice e quindi estremamente efficiente. Se un visitatore richiede un file (ad esempio una pagina HTML), in genere il file viene prelevato direttamente dalla memoria, nel caso in cui venga letto dall’unità locale. I server web statici sono disponibili da molto tempo, per cui esiste una vasta gamma di strumenti per la gestione dell'amministrazione e della sicurezza e sono perfettamente integrati con le infrastrutture di rete.

### Server di gestione dei contenuti {#content-management-servers}

![](assets/chlimage_1-4.png)

Se utilizzate un Content Management Server, come AEM, un motore di layout avanzato elabora la richiesta di un visitatore. Il motore legge il contenuto da un archivio che, insieme a stili, formati e diritti di accesso, trasforma il contenuto in un documento personalizzato in base alle esigenze e ai diritti del visitatore.

Questo consente di creare contenuti dinamici più ricchi, aumentando la flessibilità e le funzionalità del sito Web. Tuttavia, il modulo di gestione del layout richiede una potenza di elaborazione maggiore rispetto a un server statico, pertanto questa configurazione potrebbe risultare rallentata se molti visitatori utilizzano il sistema.

## Modalità di memorizzazione nella cache da parte del dispatcher {#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**Directory** cache Per il caching, il modulo Dispatcher utilizza la capacità del server Web di distribuire contenuto statico. Il dispatcher inserisce i documenti memorizzati nella directory principale del documento del server Web.

>[!NOTE]
>
>Se manca la configurazione per la memorizzazione nella cache delle intestazioni HTTP, il dispatcher memorizza solo il codice HTML della pagina e non memorizza le intestazioni HTTP. Questo può essere un problema se utilizzate codifiche diverse all'interno del sito Web, in quanto queste potrebbero andare perdute. Per abilitare la memorizzazione nella cache delle intestazioni HTTP, consultate [Configurazione della cache del dispatcher.](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)

>[!NOTE]
>
>Individuare la radice del documento del server Web su uno storage NAS (Network Attached Storage) provoca un deterioramento delle prestazioni. Inoltre, quando una radice del documento che si trova sul NAS viene condivisa tra più server Web, è possibile che si verifichino blocchi intermittenti quando vengono eseguite le azioni di replica.

>[!NOTE]
>
>Il dispatcher memorizza il documento memorizzato nella cache in una struttura uguale all'URL richiesto.
>
>Esistono limitazioni a livello di sistema operativo per la lunghezza del nome del file; ad es. se disponete di un URL con molti selettori.

### Metodi per la memorizzazione nella cache

Il Dispatcher dispone di due metodi principali per aggiornare il contenuto della cache quando vengono apportate modifiche al sito Web.

* **Aggiornamenti** contenuto rimuove le pagine che sono state modificate e i file che sono direttamente associati ad esse.
* **L'annullamento della validità** automatica invalida automaticamente le parti della cache che potrebbero non essere più aggiornate dopo un aggiornamento. ovvero contrassegna efficacemente le pagine rilevanti come non aggiornate, senza eliminare nulla.

### Aggiornamenti contenuto

In un aggiornamento del contenuto, uno o più documenti AEM vengono modificati. AEM invia una richiesta di sindacazione al dispatcher, che aggiorna di conseguenza la cache:

1. Elimina i file modificati dalla cache.
1. Vengono eliminati dalla cache tutti i file che iniziano con lo stesso handle. Ad esempio, se il file /en/index.html viene aggiornato, tutti i file che iniziano con /en/index. sono soppressi. Questo meccanismo consente di progettare siti efficienti in termini di cache, soprattutto per quanto riguarda la navigazione delle immagini.
1. *tocca* il cosiddetto **file** di stato; questo aggiorna la marca temporale del file di stato per indicare la data dell'ultima modifica.

Si osservano i seguenti punti:

* Gli aggiornamenti dei contenuti vengono in genere utilizzati insieme a un sistema di authoring che "sa" cosa deve essere sostituito.
* I file interessati da un aggiornamento del contenuto vengono rimossi ma non sostituiti immediatamente. Alla successiva richiesta di tale file, il dispatcher recupera il nuovo file dall’istanza AEM e lo inserisce nella cache, sovrascrivendo in tal modo il contenuto precedente.
* In genere, le immagini generate automaticamente che incorporano testo da una pagina vengono memorizzate in file immagine a partire dalla stessa maniglia, garantendo così l'esistenza dell'associazione per l'eliminazione. Ad esempio, potete memorizzare il testo del titolo della pagina mypage.html come immagine mypage.titlePicture.gif nella stessa cartella. In questo modo l'immagine viene eliminata automaticamente dalla cache ogni volta che la pagina viene aggiornata, in modo da essere sicuri che l'immagine rifletta sempre la versione corrente della pagina.
* Possono essere presenti diversi file di stato, ad esempio uno per cartella lingua. Se una pagina viene aggiornata, AEM cerca la cartella padre successiva contenente un file di stato e lo *tocca* .

### Annullamento automatico

L'annullamento automatico della convalida invalida automaticamente parti della cache, senza eliminare fisicamente alcun file. Ad ogni aggiornamento del contenuto, il cosiddetto file di stato viene toccato, pertanto la marca temporale corrispondente riflette l'ultimo aggiornamento del contenuto.

Il Dispatcher dispone di un elenco di file soggetti a annullamento della validità automatica. Quando viene richiesto un documento da tale elenco, il dispatcher confronta la data del documento memorizzato nella cache con la marca temporale del file di stato:

* se il documento memorizzato nella cache è più recente, il dispatcher lo restituisce.
* se è precedente, il dispatcher recupera la versione corrente dall’istanza AEM.

Anche in questo caso, occorre rilevare alcuni punti:

* L'annullamento automatico è in genere utilizzato quando le interrelazioni sono complesse, ad esempio per le pagine HTML. Queste pagine contengono collegamenti e voci di navigazione, pertanto in genere devono essere aggiornate dopo un aggiornamento del contenuto. Se avete generato automaticamente file PDF o di immagini, potete scegliere di annullare automaticamente anche questi.
* L'annullamento automatico della convalida non richiede alcuna azione da parte del dispatcher al momento dell'aggiornamento, ad eccezione del tocco del file di stato. Tuttavia, toccando il file di stato il contenuto della cache diventa automaticamente obsoleto, senza rimuoverlo fisicamente dalla cache.

## Come il dispatcher restituisce i documenti {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### Determinazione dell'oggetto caching di un documento

È possibile [definire i documenti memorizzati nella cache del Dispatcher nel file](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)di configurazione. Il Dispatcher controlla la richiesta rispetto all'elenco dei documenti memorizzabili nella cache. Se il documento non è incluso in questo elenco, il dispatcher richiede il documento dall’istanza AEM.

Il dispatcher richiede *sempre* il documento direttamente dall’istanza di AEM nei casi seguenti:

* Se l’URI della richiesta contiene il punto interrogativo "?". In genere indica una pagina dinamica, ad esempio un risultato di ricerca, che non deve essere memorizzata nella cache.
* Estensione del file mancante. Per determinare il tipo di documento (il tipo MIME) è necessario utilizzare l'estensione per il server Web.
* L'intestazione di autenticazione è impostata (può essere configurata)

>[!NOTE]
>
>I metodi GET o HEAD (per l’intestazione HTTP) sono memorizzabili nella cache dal dispatcher. Per ulteriori informazioni sul caching delle intestazioni delle risposte, consultate la sezione [Memorizzazione in cache delle intestazioni](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html) di risposta HTTP.

### Determinare se un documento è memorizzato nella cache

Il Dispatcher memorizza i file memorizzati nella cache sul server Web come se fossero parte di un sito Web statico. Se un utente richiede un documento memorizzabile nella cache, il dispatcher controlla se il documento esiste nel file system del server Web:

* se il documento è memorizzato nella cache, il dispatcher restituisce il file.
* se non è memorizzato nella cache, il dispatcher richiede il documento dall’istanza AEM.

### Determinare se un documento è aggiornato

Per verificare se un documento è aggiornato, il dispatcher esegue due passaggi:

1. Controlla se il documento è soggetto all'annullamento automatico della convalida. In caso contrario, il documento viene considerato aggiornato.
1. Se il documento è configurato per l'annullamento automatico della convalida, il dispatcher controlla se è più vecchio o più recente dell'ultima modifica disponibile. Se è precedente, il dispatcher richiede la versione corrente dall’istanza di AEM e sostituisce la versione presente nella cache.

>[!NOTE]
>
>I documenti senza **autoannullamento** rimangono nella cache finché non vengono fisicamente eliminati; ad esempio mediante un aggiornamento del contenuto sul sito Web.

## I vantaggi del bilanciamento del carico {#the-benefits-of-load-balancing}

Bilanciamento del carico è la pratica di distribuire il carico di calcolo del sito Web in diverse istanze di AEM.

![](assets/chlimage_1-7.png)

Si ottiene:

* **maggiore potenza** di elaborazione In pratica ciò significa che il dispatcher condivide le richieste di documenti tra diverse istanze di AEM. Poiché ogni istanza dispone ora di meno documenti da elaborare, i tempi di risposta sono più rapidi. Il Dispatcher conserva statistiche interne per ciascuna categoria di documenti, in modo da poter stimare il carico e distribuire le query in modo efficiente.

* **maggiore copertura** non sicura in caso di mancata ricezione delle risposte da un'istanza, il dispatcher invierà automaticamente le richieste a un'altra istanza. Pertanto, se un'istanza diventa non disponibile, l'unico effetto è un rallentamento del sito, proporzionato alla potenza di calcolo persa. Tuttavia, tutti i servizi continueranno.

* potete inoltre gestire diversi siti Web sullo stesso server Web statico.

>[!NOTE]
>
>Mentre il bilanciamento del carico viene distribuito in modo efficiente, la memorizzazione nella cache contribuisce a ridurre il carico. Pertanto, provate a ottimizzare il caching e ridurre il carico complessivo prima di impostare il bilanciamento del carico. Una buona memorizzazione nella cache può migliorare le prestazioni del sistema di bilanciamento del carico o rendere superfluo il bilanciamento del carico.

>[!CAUTION]
>
>Sebbene un singolo dispatcher sia in genere in grado di saturare la capacità delle istanze di pubblicazione disponibili, per alcune rare applicazioni può essere utile bilanciare ulteriormente il carico tra due istanze del dispatcher. Le configurazioni con più dispatcher devono essere considerate attentamente, poiché un ulteriore dispatcher incrementerà il carico sulle istanze di pubblicazione disponibili e potrà facilmente diminuire le prestazioni nella maggior parte delle applicazioni.

## Modalità di esecuzione del bilanciamento del carico da parte del dispatcher {#how-the-dispatcher-performs-load-balancing}

### Statistiche di prestazioni

Il dispatcher conserva statistiche interne sulla velocità con cui ogni istanza di AEM elabora i documenti. In base a questi dati, il Dispatcher calcola quale istanza fornirà il tempo di risposta più rapido quando risponderà a una richiesta, e quindi riserva il tempo di calcolo necessario su tale istanza.

Diversi tipi di richieste possono avere tempi medi di completamento diversi, pertanto il dispatcher consente di specificare le categorie dei documenti. Questi vengono quindi considerati quando si calcolano le stime temporali. Ad esempio, potete fare una distinzione tra pagine HTML e immagini, poiché i tempi di risposta tipici potrebbero essere molto diversi.

Se si utilizza una funzione di ricerca elaborata, è possibile creare una nuova categoria per le query di ricerca. Questo consente al Dispatcher di inviare le query di ricerca all'istanza che risponde più velocemente. Questo impedisce a un’istanza più lenta di rimanere in sospeso quando riceve diverse query di ricerca "costose", mentre alle altre richieste "più economiche".

### Contenuto personalizzato (connessioni permanenti)

Le connessioni permanenti garantiscono che i documenti per un utente siano composti tutti sulla stessa istanza di AEM. Ciò è importante se utilizzate pagine personalizzate e dati di sessione. I dati vengono memorizzati nell'istanza, per cui le richieste successive dello stesso utente devono tornare all'istanza oppure i dati vanno persi.

Poiché le connessioni fisse limitano la capacità del dispatcher di ottimizzare le richieste, è consigliabile utilizzarle solo quando necessario. Potete specificare la cartella che contiene i documenti "fissi", in modo che tutti i documenti in tale cartella siano composti sulla stessa istanza per ogni utente.

>[!NOTE]
>
>Per la maggior parte delle pagine che utilizzano connessioni fisse, è necessario disattivare la memorizzazione nella cache, altrimenti la pagina avrà lo stesso aspetto per tutti gli utenti, indipendentemente dal contenuto della sessione.
>
>Per *alcune* applicazioni, è possibile utilizzare sia connessioni fissi che cache; ad esempio, se viene visualizzato un modulo che scrive dati per la sessione.

## Utilizzo di più dispatcher {#using-multiple-dispatchers}

In configurazioni complesse, puoi utilizzare più dispatcher. Ad esempio, potete utilizzare:

* un Dispatcher per pubblicare un sito Web sulla Intranet
* un secondo dispatcher, con un indirizzo diverso e con impostazioni di protezione diverse, per pubblicare lo stesso contenuto su Internet.

In tal caso, accertatevi che ogni richiesta passi attraverso un solo Dispatcher. Un dispatcher non gestisce le richieste provenienti da un altro dispatcher. Assicuratevi pertanto che entrambi i dispatcher accedano direttamente al sito Web AEM.

## Utilizzo di Dispatcher con un CDN {#using-dispatcher-with-a-cdn}

Una rete di distribuzione dei contenuti (CDN), come Akamai Edge Delivery o Amazon Cloud Front, distribuisce i contenuti da una posizione vicina all'utente finale. Per questo

* velocizza i tempi di risposta per gli utenti finali
* scarica i server

Come componente di infrastruttura HTTP, una rete CDN funziona come Dispatcher: quando un nodo CDN riceve una richiesta, invia la richiesta dalla cache, se possibile (la risorsa è disponibile nella cache ed è valida). In caso contrario, il server successivo più vicino può recuperare la risorsa e memorizzarla nella cache per ulteriori richieste, se appropriato.

Il "server più vicino successivo" dipende dalla configurazione specifica. Ad esempio, in una configurazione di Akamai la richiesta può seguire il percorso seguente:

* Il nodo Edge di Akamai
* Livello Akamai Midgres
* Il firewall
* Il tuo sistema di bilanciamento del carico
* Dispatcher
* AEM

Nella maggior parte dei casi, Dispatcher è il server successivo che potrebbe distribuire il documento da una cache e influenzare le intestazioni di risposta restituite al server CDN.

## Controllo di una cache CDN {#controlling-a-cdn-cache}

Esistono diversi modi per controllare per quanto tempo una CDN memorizzerà nella cache una risorsa prima che venga recuperata dal Dispatcher.

1. Configurazione esplicita\
   Configura, per quanto tempo particolari risorse vengono memorizzate nella cache della CDN, a seconda del tipo di mime, dell’estensione, del tipo di richiesta, ecc.

1. Intestazioni di scadenza e controllo cache\
   La maggior parte dei CDN rispetterà le intestazioni `Expires:` e `Cache-Control:` HTTP se inviati dal server upstream. Questo può essere ottenuto, ad esempio utilizzando il modulo Apache [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) .

1. Annullamento manuale\
   I CDN consentono di rimuovere risorse dalla cache tramite interfacce Web.
1. Annullamento basato su API\
   La maggior parte dei CDN offre anche un REST e/o SOAP API che consente di rimuovere le risorse dalla cache.

In una tipica configurazione di AEM, la configurazione per estensione e/o per percorso, ottenibile tramite i punti 1 e 2 di cui sopra, offre la possibilità di impostare periodi di memorizzazione nella cache ragionevoli per risorse spesso utilizzate che non vengono modificate spesso, come immagini di progettazione e librerie client. Quando vengono distribuite nuove versioni, in genere è necessaria un'annullamento della validità manuale.

Se questo approccio viene utilizzato per memorizzare nella cache il contenuto gestito, significa che le modifiche al contenuto sono visibili solo agli utenti finali una volta scaduto il periodo di memorizzazione nella cache configurato e che il documento viene recuperato dal dispatcher.

Per un controllo con granulosità maggiore, l'annullamento della validità API consente di annullare la validità della cache di un CDN in quanto la cache del dispatcher viene invalidata. In base all’API CDN, potete implementare [ContentBuilder](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/ContentBuilder.html) e [TransportHandler](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/TransportHandler.html) personalizzati (se l’API non è basata su REST) e impostare un agente di replica che li utilizzerà per annullare la validità della cache CDN.

>[!NOTE]
>
>Consultate anche [AEM (CQ) Dispatcher Security (Protezione dispatcher) e CDN+Memorizzazione nella cache](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015) del browser e presentazione registrata nella cache del [dispatcher](https://docs.adobe.com/content/ddc/en/gems/dispatcher-caching---new-features-and-optimizations.html).

## Utilizzo di un dispatcher con un server Author {#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>se utilizzate [AEM con l'interfaccia](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html) touch, **non** includete nella cache il contenuto dell'istanza dell'autore. Se la memorizzazione nella cache è stata abilitata per l’istanza di creazione, è necessario disattivarla ed eliminare il contenuto della directory della cache. Per disattivare il caching, è necessario modificare il `author_dispatcher.any` file e modificare la `/rule` proprietà della `/cache` sezione come segue:

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

È possibile utilizzare un dispatcher davanti a un’istanza di creazione per migliorare le prestazioni di authoring. Per configurare un dispatcher di authoring, effettuate le seguenti operazioni:

1. Installare un dispatcher in un server Web (potrebbe essere un server Web Apache o IIS, vedere [Installazione del dispatcher](dispatcher-install.md)).
1. È possibile sottoporre a test il dispatcher appena installato rispetto a un'istanza di pubblicazione AEM funzionante, per verificare che sia stata raggiunta un'installazione corretta della linea di base.
1. Ora accertatevi che il dispatcher sia in grado di connettersi all'istanza di creazione tramite TCP/IP.
1. Sostituire il file dispatcher.any di esempio con il file author_dispatcher.any fornito con il download [del](release-notes.md#downloads)dispatcher.
1. Aprite il modulo `author_dispatcher.any` in un editor di testo e apportate le seguenti modifiche:

   1. Modificate la sezione `/hostname` e `/port` `/renders` la sezione in modo che puntino all’istanza di creazione.
   1. Modificate la `/docroot` `/cache` sezione in modo che punti a una directory della cache. Se utilizzi [AEM con l’interfaccia](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html)touch, consulta l’avviso qui sopra.
   1. Salvate le modifiche.

1. Eliminate tutti i file esistenti nella directory `/cache` &gt; `/docroot` configurata sopra.
1. Riavviate il server Web.

>[!NOTE]
>
>Con la `author_dispatcher.any` configurazione fornita, quando installate un pacchetto di funzioni, un hotfix o un pacchetto di codice dell’applicazione CQ5 che interessa qualsiasi contenuto in `/libs` oppure `/apps` dovete eliminare i file memorizzati nella cache di tali directory per fare in modo che la prossima volta che vengono richiesti, i file aggiornati di recente vengano recuperati e non quelli memorizzati nella cache.

>[!CAUTION]
>
>Se avete utilizzato il dispatcher di autori precedentemente configurato e avete attivato un agente *di* eliminazione del dispatcher, effettuate le seguenti operazioni:

1. Eliminate o disattivate l’agente di eliminazione del dispatcher di **autori** nell’istanza di creazione di AEM.
1. Ripetete la configurazione del dispatcher di autori seguendo le nuove istruzioni riportate sopra.

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
