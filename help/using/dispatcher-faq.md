---
title: Problemi principali del dispatcher
seo-title: Problemi principali per Dispatcher AEM
description: Problemi principali per Dispatcher AEM
seo-description: Problemi principali per Adobe AEM Dispatcher
translation-type: tm+mt
source-git-commit: 76cffbfb616cd5601aed36b7076f67a2faf3ed3b

---


# Domande frequenti sul dispatcher di AEM

![Configurazione del dispatcher](assets/CQDispatcher_workflow_v2.png)

## Introduzione

### Cos&#39;è il dispatcher?

Il Dispatcher è uno strumento di cache e/o bilanciamento del carico di Adobe Experience Manager che contribuisce a realizzare un ambiente di authoring Web veloce e dinamico. Per il caching, il dispatcher funziona come parte di un server HTTP, ad esempio Apache, allo scopo di memorizzare (o «memorizzare nella cache») il contenuto del sito Web statico e di accedere al motore di layout del sito Web il minor numero di volte possibile. In un ruolo di bilanciamento del carico, il dispatcher distribuisce richieste (load) tra diverse istanze di AEM (rendering).

Per il caching, il modulo Dispatcher utilizza la capacità del server Web di distribuire il contenuto statico. Il dispatcher colloca i documenti memorizzati nella cache nella directory principale del documento del server Web.

### In che modo il dispatcher esegue il caching?

Il dispatcher utilizza la capacità del server Web di distribuire il contenuto statico. Il dispatcher memorizza i documenti memorizzati nella cache nella directory principale del documento del server Web. Il dispatcher dispone di due metodi principali per aggiornare il contenuto della cache quando vengono apportate modifiche al sito Web.

* **Aggiornamenti** dei contenuti rimuove le pagine modificate e i file direttamente associati.
* **Annulla validità automatica** invalida automaticamente le parti della cache che potrebbero non essere aggiornate dopo un aggiornamento. Ad esempio, contrassegna le pagine pertinenti come non aggiornate, senza eliminare nulla.

### Quali sono i vantaggi del bilanciamento del carico?

Il bilanciamento del carico distribuisce le richieste degli utenti (load) in diverse istanze AEM. L&#39;elenco seguente descrive i vantaggi per il bilanciamento del carico:

* **Potenza elaborazione aumentata**: In pratica, il dispatcher condivide le richieste del documento tra più istanze di AEM. Poiché ogni istanza dispone di un numero inferiore di documenti da elaborare, il tempo di risposta è più rapido. Il dispatcher mantiene statistiche interne per ogni categoria di documenti, in modo da stimare il carico e distribuire le query in modo efficiente.
* **Copertura sicura non riuscita**: Se il dispatcher non riceve le risposte da un&#39;istanza, ripeterà automaticamente le richieste a una delle altre istanze. Pertanto, se un&#39;istanza diventa non disponibile, l&#39;unico effetto è un rallentamento del sito, proporzionali alla potenza computazionale persa.

>[!NOTE]
>
>Per ulteriori dettagli, consultate la pagina Panoramica [del dispatcher](dispatcher.md)

## Installazione e configurazione

### Da dove si scarica il modulo Dispatcher?

Puoi scaricare il modulo più recente del dispatcher dalla pagina [Note](release-notes.md) sulla versione del dispatcher.

### Come si installa il modulo Dispatcher?

Refer to the [Installing Dispatcher](dispatcher-install.md) page

### Come posso configurare il modulo Dispatcher?

Consulta [la](dispatcher-configuration.md) pagina Configurazione del dispatcher.

### Come posso configurare il dispatcher per l&#39;istanza di authoring?

Consulta [Utilizzo del dispatcher con un&#39;istanza autore](dispatcher.md#using-a-dispatcher-with-an-author-server) per i passaggi dettagliati.

### Come posso configurare il dispatcher con più domini?

Puoi configurare CQ Dispatcher con più domini, a condizione che i domini soddisfino le condizioni seguenti:

* Il contenuto Web per entrambi i domini è memorizzato in un unico archivio AEM
* I file nella cache del dispatcher possono essere invalidati separatamente per ogni dominio

Leggi [Utilizzo del dispatcher con più domini](dispatcher-domains.md) per ulteriori dettagli.

### Come posso configurare il dispatcher, in modo che tutte le richieste di un utente vengano indirizzate alla stessa istanza Pubblica?

Potete [utilizzare la funzione di connessioni](dispatcher-configuration.md#identifying-a-sticky-connection-folder-stickyconnectionsfor) fisso, che garantisce che tutti i documenti per un utente vengano elaborati sulla stessa istanza di AEM. Questa funzione è importante se usi pagine personalizzate e dati di sessione. I dati vengono memorizzati nell&#39;istanza. Pertanto, le richieste successive dello stesso utente devono restituire l&#39;istanza o i dati vanno persi.

Poiché le connessioni fisso limitano la capacità del dispatcher di ottimizzare le richieste, è consigliabile utilizzare questo approccio solo se necessario. Potete specificare la cartella che contiene i documenti «fisso», in modo che tutti i documenti della cartella vengano elaborati sulla stessa istanza per un utente.

### È possibile utilizzare collegamenti fissi e memorizzazione nella cache?

Per la maggior parte delle pagine che utilizzano connessioni fisse, disattivare la memorizzazione nella cache. In caso contrario, la stessa istanza della pagina viene visualizzata a tutti gli utenti, indipendentemente dal contenuto della sessione.

Per alcune applicazioni è possibile utilizzare sia connessioni che caching. Ad esempio, se si visualizza un modulo che scrive dati in una sessione, è possibile utilizzare in parallelo connessioni fissi e memorizzazione nella cache.

### Un dispatcher e un&#39;istanza AEM Publish risiedono sullo stesso computer fisico?

Sì, se il computer è sufficientemente potente. Tuttavia, si consiglia di configurare il dispatcher e l&#39;istanza AEM Publish su computer diversi.

Generalmente, l&#39;istanza Pubblica risiede all&#39;interno del firewall e il dispatcher risiede nella rete perimetrale. Se si decide di avere sia l&#39;istanza Pubblica che il dispatcher sullo stesso computer fisico, assicurarsi che le impostazioni del firewall non consentano l&#39;accesso diretto all&#39;istanza Pubblica da reti esterne.

### Posso memorizzare nella cache solo file con estensioni specifiche?

Sì. Ad esempio, se desiderate memorizzare nella cache solo i file GIF, specificate *. gif nella sezione della cache del dispatcher. Qualsiasi file di configurazione.

### Come si eliminano i file dalla cache?

Potete eliminare i file dalla cache utilizzando una richiesta HTTP. Quando viene ricevuta la richiesta HTTP, Dispatcher elimina i file dalla cache. Il dispatcher memorizza nuovamente i file solo quando riceve una richiesta client per la pagina. L&#39;eliminazione di file memorizzati nella cache in questo modo è opportuna per i siti Web che non riceveranno richieste simultanee per la stessa pagina.

La richiesta HTTP ha la sintassi seguente:

```
POST /dispatcher/invalidate.cache HTTP/1.1
CQ-Action: Activate
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher Elimina i file e le cartelle memorizzati nella cache con nomi corrispondenti al valore dell&#39;intestazione Handle CQ. Ad esempio, una maniglia CQ di `/content/geomtrixx-outdoors/en` corrisponde ai seguenti elementi:

Tutti i file (di qualsiasi estensione di file) denominati en nella directory
geometrixx-outdoors Qualsiasi directory denominata `_jcr_content` sotto la directory en (che, se esiste, contiene rendering memorizzati nella cache dei nodi della pagina)
La directory en verrà eliminata solo se è `CQ-Action``Delete` o `Deactivate`.

Per ulteriori dettagli su questo argomento, consultate [Annullamento manuale della cache del dispatcher](page-invalidate.md).

### Come si implementa caching sensibile alle autorizzazioni?

Vedere la pagina [Memorizzazione nella cache contenuto](permissions-cache.md) protetto.

### Come si proteggono le comunicazioni tra le istanze di Dispatcher e CQ?

Consultate le [pagine Elenco di controllo](security-checklist.md) della sicurezza [del dispatcher e Elenco di controllo](https://helpx.adobe.com/experience-manager/6-4/sites/administering/using/security-checklist.html) AEM.

### Edizione dispatcher `jcr:content` modificata in `jcr%3acontent`

**Domanda**: Di recente è stato riscontrato un problema a livello di dispatcher, in cui una chiamata ajax con alcuni repository di CQ risultava `jcr:content` in essa e la codifica `jcr%3acontent` risultava errata.

**Risposta**: Utilizzate `ResourceResolver.map()` il metodo per ottenere un URL descrittivo da utilizzare/emettere richieste, nonché per risolvere il problema di caching con Dispatcher. Il metodo map () codifica i `:` due punti ai caratteri di sottolineatura e il metodo resolve () li decodifica in formato leggibile JCR JCR. È necessario utilizzare il metodo map () per generare l&#39;URL utilizzato nella chiamata Ajax.

Leggi ulteriormente: [https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling)

## Svuotare il dispatcher

### Come posso configurare gli agenti di svuotamento del dispatcher su un&#39;istanza Pubblica?

Consultate la pagina [Replica](https://helpx.adobe.com/content/help/en/experience-manager/6-4/sites/deploying/using/replication.html#ConfiguringyourReplicationAgents) .

### Come posso risolvere i problemi di cancellazione del dispatcher?

[Consultate questo articolo](https://helpx.adobe.com/content/help/en/experience-manager/kb/troubleshooting-dispatcher-flushing-issues.html) sulla risoluzione dei problemi che risponde alle seguenti domande:

* Come si effettua il debug di una situazione in cui non viene salvato alcun contenuto nella cache del dispatcher?
* Come si esegue il debug di una situazione in cui i file della cache non vengono aggiornati?
* Come si esegue il debug di una situazione in cui nulla correlato all&#39;eliminazione del dispatcher funziona?

Se le operazioni Elimina causano lo svuotamento del dispatcher, [utilizzate la soluzione descritta in questo post di blog della community di Sensei Martin](https://mkalugin-cq.blogspot.in/2012/04/i-have-been-working-on-following.html).

### Come si eliminano le risorse DAM dalla cache del dispatcher?

Potete utilizzare la funzione «replica catena». Con questa funzione abilitata, l&#39;agente di svuotamento del dispatcher invia una richiesta di svuotamento quando viene ricevuta una replica dall&#39;autore.

Per attivarla:

1. [Segui i passaggi qui riportati](page-invalidate.md#invalidating-dispatcher-cache-from-a-publishing-instance) per creare agenti di eliminazione durante la pubblicazione
1. Accedi a ciascuna configurazione dell&#39;agente e nella scheda **Triggers** seleziona la **casella In ricezione** .

## Varie

In che modo il dispatcher determina se un documento è aggiornato?
Per determinare se un documento è aggiornato, il Dispatcher esegue le azioni seguenti:

Controlla se il documento è soggetto a annullamento automatico del documento. In caso contrario, il documento viene considerato aggiornato.
Se il documento è configurato per la cancellazione automatica, il dispatcher verifica se è precedente o più recente dell&#39;ultima modifica disponibile. Se è precedente, il dispatcher richiede la versione corrente dall&#39;istanza AEM e sostituisce la versione nella cache.

### In che modo il dispatcher restituisce i documenti?

You can define whether the Dispatcher caches a document by using the [Dispatcher configuration](dispatcher-configuration.md) file, `dispatcher.any`. Il dispatcher verifica la richiesta rispetto all&#39;elenco di documenti CACHABLE. Se il documento non è presente in questo elenco, il dispatcher richiede il documento dall&#39;istanza AEM.

La `/rules` proprietà controlla i documenti memorizzati nella cache in base al percorso del documento. A prescindere `/rules` dalla proprietà, Dispatcher non memorizza mai nella cache un documento nelle seguenti circostanze:

* Se l&#39;URI di richiesta contiene un punto `(?)`interrogativo.
* In genere indica una pagina dinamica, ad esempio un risultato di ricerca che non deve essere memorizzato nella cache.
* L&#39;estensione del file è mancante.
* Il server Web richiede l&#39;estensione per determinare il tipo di documento (tipo MIME).
* L&#39;intestazione di autenticazione è impostata (può essere configurata)
* Se l&#39;istanza AEM risponde con le seguenti intestazioni:
   * no-cache
   * no-store
   * must-revalidate

Il Dispatcher memorizza i file memorizzati nella cache sul server Web come se fossero parte di un sito Web statico. Se un utente richiede un documento memorizzato nella cache, il dispatcher verifica se il documento esiste nel file system del server Web. In tal caso, il dispatcher restituisce i documenti. In caso contrario, il dispatcher richiede il documento dall&#39;istanza AEM.

>[!NOTE]
>
>I metodi GET o HEAD (per i metodi dell&#39;intestazione HTTP) possono essere memorizzati nella cache dal dispatcher. Per ulteriori informazioni sul caching delle intestazioni della risposta, consultate la [sezione Intestazioni](dispatcher-configuration.md#caching-http-response-headers) risposta HTTP.

### Posso implementare più dispatcher in una configurazione?

Sì. In tali casi, assicuratevi che entrambi i dispatcher possano accedere direttamente al sito Web AEM. Un dispatcher non può gestire le richieste provenienti da un altro dispatcher.

