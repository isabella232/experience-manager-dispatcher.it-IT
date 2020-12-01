---
title: Problemi principali del dispatcher
seo-title: Problemi principali per AEM Dispatcher
description: Problemi principali per AEM Dispatcher
seo-description: Problemi principali per  Adobe AEM Dispatcher
translation-type: tm+mt
source-git-commit: eed7c3f77ec64f2e7c5cfff070ef96108886a059
workflow-type: tm+mt
source-wordcount: '1644'
ht-degree: 13%

---


# Domande frequenti sui problemi principali del dispatcher AEM

![Configurazione di Dispatcher](assets/CQDispatcher_workflow_v2.png)

## Introduzione

### Cos&#39;è il Dispatcher?

Dispatcher è uno strumento di cache e/o bilanciamento del carico Adobe Experience Manager che consente di realizzare un ambiente di authoring Web veloce e dinamico. Per il caching, il Dispatcher funziona come parte di un server HTTP, come Apache, con lo scopo di memorizzare (o &quot;caching&quot;) la maggior parte del contenuto statico del sito Web e di accedere il più raramente possibile al motore di layout del sito Web. In un ruolo di bilanciamento del carico, il dispatcher distribuisce le richieste degli utenti (caricate) tra istanze AEM diverse (rendering).

Per il caching, il modulo Dispatcher utilizza la capacità del server Web di distribuire contenuto statico. Il Dispatcher inserisce i documenti memorizzati nella directory principale del documento del server Web.

### Come viene eseguito il caching del dispatcher?

Il dispatcher utilizza la capacità del server Web di distribuire contenuti statici. Il dispatcher memorizza i documenti memorizzati nella cache nella directory principale del documento del server Web. In Dispatcher sono disponibili due metodi principali per aggiornare il contenuto della cache quando vengono apportate modifiche al sito web.

* **Aggiornamenti del contenuto**: le pagine modificate e i file che sono direttamente associati ad esse vengono rimossi.
* **Annullamento automatico della validità**: le parti della cache che potrebbero risultare obsolete dopo un aggiornamento vengono invalidate automaticamente, Ad esempio, contrassegna efficacemente le pagine rilevanti come non aggiornate, senza eliminare nulla.

### Quali sono i vantaggi del bilanciamento del carico?

Il bilanciamento del carico distribuisce le richieste degli utenti (carico) in diverse istanze AEM.L&#39;elenco seguente descrive i vantaggi del bilanciamento del carico:

* **Maggiore potenza** di elaborazione: In pratica ciò significa che il Dispatcher condivide le richieste di documenti tra più istanze di AEM. Poiché ogni istanza dispone di un numero inferiore di documenti da elaborare, i tempi di risposta sono più rapidi. Dispatcher mantiene statistiche interne per ogni categoria di documenti, in modo da poter stimare il carico e distribuire le query in modo efficiente.
* **Maggiore copertura** non sicura: Se il Dispatcher non riceve risposte da un&#39;istanza, le richieste verranno inviate automaticamente a un&#39;altra istanza. Pertanto, se un’istanza risulta non disponibile, l’unico effetto è un rallentamento del sito, proporzionato alla potenza di elaborazione persa,

>[!NOTE]
>
>Per ulteriori dettagli, vedere la pagina [Panoramica del dispatcher](dispatcher.md)

## Installazione e configurazione

### Da dove si scarica il modulo Dispatcher?

È possibile scaricare l&#39;ultimo modulo del dispatcher dalla pagina [Note sulla versione del dispatcher](release-notes.md).

### Come si installa il modulo Dispatcher?

Fare riferimento alla pagina [Installazione del dispatcher](dispatcher-install.md)

### Come si configura il modulo Dispatcher?

Vedere la pagina [Configuring Dispatcher](dispatcher-configuration.md).

### Come si configura il dispatcher per l’istanza di creazione?

Per i passaggi dettagliati, vedere [Utilizzo del dispatcher con un&#39;istanza Author](dispatcher.md#using-a-dispatcher-with-an-author-server).

### Come si configura il dispatcher con più domini?

È possibile configurare il dispatcher CQ con più domini, purché i domini soddisfino le seguenti condizioni:

* Il contenuto Web di entrambi i domini è memorizzato in un unico archivio AEM
* I file nella cache del dispatcher possono essere invalidati separatamente per ciascun dominio

Per ulteriori informazioni, consultare [Utilizzo del dispatcher con più domini](dispatcher-domains.md).

### Come posso configurare il dispatcher in modo che tutte le richieste di un utente vengano instradate nella stessa istanza di pubblicazione?

È possibile utilizzare la funzione [connessioni fisse](dispatcher-configuration.md#identifying-a-sticky-connection-folder-stickyconnectionsfor), che assicura che tutti i documenti per un utente vengano elaborati sulla stessa istanza di AEM. Questa funzione è importante se utilizzate pagine personalizzate e dati di sessione. I dati vengono memorizzati nell&#39;istanza. Pertanto, le richieste successive dello stesso utente devono tornare a tale istanza o i dati vanno persi.

Poiché le connessioni fisse limitano la capacità del dispatcher di ottimizzare le richieste, è consigliabile utilizzare questo approccio solo quando necessario. Potete specificare la cartella che contiene i documenti &quot;fissi&quot;, garantendo in tal modo che tutti i documenti in tale cartella vengano elaborati nella stessa istanza per un utente.

### È possibile utilizzare collegamenti fissi e caching in tandem?

Per la maggior parte delle pagine che utilizzano collegamenti fissi, è necessario disattivare la memorizzazione nella cache. In caso contrario, la stessa istanza della pagina viene visualizzata a tutti gli utenti, indipendentemente dal contenuto della sessione.

Per alcune applicazioni, è possibile utilizzare sia connessioni fisse che cache. Ad esempio, se si visualizza un modulo che scrive dati in una sessione, è possibile utilizzare connessioni permanenti e memorizzazione nella cache in parallelo.

### Un dispatcher e un&#39;istanza di AEM Publish possono risiedere sullo stesso computer fisico?

Sì, se la macchina è sufficientemente potente. Tuttavia, si consiglia di impostare il dispatcher e l’istanza AEM Publish su computer diversi.

Solitamente, l’istanza Pubblica risiede all’interno del firewall e il Dispatcher risiede nella rete perimetrale. Se decidete di disporre sia dell’istanza Pubblica che del Dispatcher sullo stesso computer fisico, accertatevi che le impostazioni del firewall vietino l’accesso diretto all’istanza Pubblica dalle reti esterne.

### È possibile memorizzare nella cache solo i file con estensioni specifiche?

Sì. Ad esempio, se desiderate memorizzare nella cache solo i file GIF, specificate *.gif nella sezione cache del file di configurazione dispatcher.any.

### Come si eliminano i file dalla cache?

Potete eliminare i file dalla cache utilizzando una richiesta HTTP. Quando viene ricevuta la richiesta HTTP, il dispatcher elimina i file dalla cache. Il dispatcher memorizza nuovamente i file nella cache solo quando riceve una richiesta client per la pagina. L’eliminazione dei file memorizzati nella cache in questo modo è appropriata per i siti Web che probabilmente non riceveranno richieste simultanee per la stessa pagina.

La richiesta HTTP presenta la sintassi seguente:

```
POST /dispatcher/invalidate.cache HTTP/1.1
CQ-Action: Activate
CQ-Handle: path-pattern
Content-Length: 0
```

Il dispatcher elimina i file e le cartelle memorizzati nella cache con nomi che corrispondono al valore dell’intestazione CQ-Handle. Ad esempio, un CQ-Handle di `/content/geomtrixx-outdoors/en` corrisponde ai seguenti elementi:

Tutti i file (di qualsiasi estensione file) denominati en nella directory geometrixx-outdoors
Qualsiasi directory denominata `_jcr_content` sotto la directory en (che, se esiste, contiene rendering memorizzati nella cache dei sottonodi della pagina)
La directory en verrà eliminata solo se `CQ-Action` è `Delete` o `Deactivate`.

Per ulteriori dettagli su questo argomento, vedere [Invalidazione manuale della cache del dispatcher](page-invalidate.md).

### Come si implementa il caching sensibile alle autorizzazioni?

Vedere la pagina [Memorizzazione nella cache di contenuto protetto](permissions-cache.md).

### Come posso proteggere le comunicazioni tra le istanze Dispatcher e CQ?

Vedere le pagine [Elenco di controllo della protezione del dispatcher](security-checklist.md) e [AEM Elenco di controllo della protezione](https://helpx.adobe.com/experience-manager/6-4/sites/administering/using/security-checklist.html).

### Problema del dispatcher `jcr:content` modificato in `jcr%3acontent`

**Domanda**: Di recente è stato riscontrato un problema a livello di dispatcher, a causa del quale una delle chiamate ajax che otteneva alcuni dati dall&#39;archivio CQ era  `jcr:content` in esso e che venivano codificati per  `jcr%3acontent` generare un set di risultati errato.

**Risposta**: Utilizzate  `ResourceResolver.map()` il metodo per ottenere un URL &quot;descrittivo&quot; da utilizzare/ricevere richieste inviate da e anche per risolvere il problema di caching con Dispatcher. Il metodo map() codifica i due punti `:` in caratteri di sottolineatura e il metodo resolve() li decodifica nel formato leggibile SLING JCR. È necessario utilizzare il metodo map() per generare l&#39;URL utilizzato nella chiamata Ajax.

Ulteriori informazioni: [https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling)

## Svuotare il dispatcher

### Come si configurano gli agenti di scaricamento del dispatcher in un’istanza di pubblicazione?

Vedere la pagina [Replication](https://helpx.adobe.com/content/help/en/experience-manager/6-4/sites/deploying/using/replication.html#ConfiguringyourReplicationAgents).

### Come si risolvono i problemi di cancellazione del Dispatcher?

[Consultare questo ](https://helpx.adobe.com/content/help/en/experience-manager/kb/troubleshooting-dispatcher-flushing-issues.html) articolo per la risoluzione dei problemi che risponde alle seguenti domande:

* Come posso eseguire il debug di una situazione in cui non viene salvato alcun contenuto nella cache del dispatcher?
* Come si esegue il debug di una situazione in cui i file della cache non vengono aggiornati?
* Come si esegue il debug di una situazione in cui non funziona nulla correlato allo scarico del dispatcher?

Se le operazioni Elimina causano lo scaricamento del dispatcher, [utilizzare la soluzione alternativa in questo post del blog community di Sensei Martin](https://mkalugin-cq.blogspot.in/2012/04/i-have-been-working-on-following.html).

### Come si scaricano le risorse DAM dalla cache del dispatcher?

È possibile utilizzare la funzione di &quot;replica a catena&quot;.  Quando questa funzione è attivata, l&#39;agente di eliminazione del dispatcher invia una richiesta di eliminazione quando l&#39;autore riceve una replica.

Per attivarla:

1. [Seguire i passaggi ](page-invalidate.md#invalidating-dispatcher-cache-from-a-publishing-instance) per creare agenti di scarico al momento della pubblicazione
1. Andate alla configurazione di ciascun agente e, nella scheda **Triggers**, selezionate la casella **Su ricezione**.

## Varie

In che modo il dispatcher determina se un documento è aggiornato?
Per determinare se un documento è aggiornato, il dispatcher esegue le azioni seguenti:

Verifica se il documento è soggetto ad annullamento automatico della validità. Se non lo è, il documento viene considerato aggiornato.
Se il documento è configurato per l’annullamento automatico della validità, Dispatcher controlla se è più o meno recente dell’ultima modifica disponibile. Se è meno recente, Dispatcher richiede la versione corrente dall’istanza di AEM e sostituisce la versione nella cache.

### In che modo il dispatcher restituisce i documenti?

È possibile definire se il dispatcher memorizza nella cache un documento utilizzando il file [Dispatcher configuration](dispatcher-configuration.md), `dispatcher.any`. Dispatcher confronta la richiesta con l’elenco dei documenti memorizzabili in cache. Se il documento non è incluso in questo elenco, Dispatcher richiede il documento dall’istanza di AEM.

La proprietà `/rules` controlla quali documenti vengono memorizzati nella cache in base al percorso del documento. Indipendentemente dalla proprietà `/rules`, il dispatcher non memorizza mai nella cache un documento nelle seguenti circostanze:

* Se l&#39;URI della richiesta contiene un punto interrogativo `(?)`.
* In genere indica una pagina dinamica, ad esempio un risultato di ricerca che non deve essere memorizzato nella cache.
* Se manca l’estensione del file.
* Il server web ha bisogno dell’estensione per determinare il tipo di documento (tipo MIME).
* Se l’intestazione di autenticazione è impostata (configurabile).
* Se l&#39;istanza AEM risponde con le seguenti intestazioni:
   * nessuna cache
   * no-store
   * must-revalidate

Il dispatcher memorizza i file memorizzati nella cache sul server Web come se fossero parte di un sito Web statico. Se un utente richiede un documento memorizzato nella cache, il dispatcher controlla se il documento esiste nel file system del server Web. In tal caso, il dispatcher restituisce i documenti. In caso contrario, il dispatcher richiede il documento dall&#39;istanza AEM.

>[!NOTE]
>
>Dispatcher può memorizzare in cache i metodi GET o HEAD (per l’intestazione HTTP). Per ulteriori informazioni sul caching delle intestazioni delle risposte, consultate la sezione [Memorizzazione nella cache delle intestazioni di risposta HTTP](dispatcher-configuration.md#caching-http-response-headers).

### Posso implementare più dispatcher in una configurazione?

Sì. In tali casi, assicurarsi che entrambi i Dispatcher possano accedere direttamente al sito Web AEM. Un dispatcher non è in grado di gestire le richieste provenienti da un altro dispatcher.

