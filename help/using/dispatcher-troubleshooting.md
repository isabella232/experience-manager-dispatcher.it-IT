---
title: Risoluzione dei problemi di Dispatcher
seo-title: Troubleshooting AEM Dispatcher Problems
description: Scopri come risolvere i problemi di Dispatcher.
seo-description: Learn to troubleshoot AEM Dispatcher issues.
uuid: 9c109a48-d921-4b6e-9626-1158cebc41e7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: a612e745-f1e6-43de-b25a-9adcaadab5cf
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: 26c8edbb142297830c7c8bd068502263c9f0e7eb
workflow-type: tm+mt
source-wordcount: '560'
ht-degree: 43%

---

# Risoluzione dei problemi di Dispatcher {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM, tuttavia la documentazione di Dispatcher è incorporata nella documentazione di AEM. Utilizza sempre la documentazione di Dispatcher incorporata nella documentazione della più recente versione di AEM.
>
>Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

>[!NOTE]
>
>Controlla la [Knowledge Base di Dispatcher](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html), [Risoluzione dei problemi di scaricamento del dispatcher](https://experienceleague.adobe.com/search.html?lang=en#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager]) e [Domande frequenti sui principali problemi di Dispatcher](dispatcher-faq.md) per ulteriori informazioni.

## Controlla la configurazione di base {#check-the-basic-configuration}

Come sempre, i primi passaggi consistono nel controllare la configurazione di base:

* [Verifica il funzionamento di base](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* Controlla tutti i file di registro per il tuo server web e Dispatcher. Se necessario, aumenta il `loglevel` utilizzato per il Dispatcher [registrazione](/help/using/dispatcher-configuration.md#logging).

* [Verifica la configurazione](/help/using/dispatcher-configuration.md):

   * Hai più Dispatcher?

      * Hai stabilito quale Dispatcher gestisce il Sito Web/la pagina che stai esaminando?
   * Hai implementato i filtri?

      * Questi filtri stanno influenzando la questione su cui stai indagando?


## Strumenti di diagnostica IIS {#iis-diagnostic-tools}

IIS fornisce vari strumenti di trace, a seconda della versione effettiva:

* IIS 6 - È possibile scaricare e configurare gli strumenti di diagnostica IIS
* IIS 7 - Il tracciamento è completamente integrato

Questi strumenti possono aiutarti a monitorare l’attività.

## Impossibile trovare IIS e 404 {#iis-and-not-found}

Quando si utilizza IIS, è possibile che `404 Not Found` essere restituiti in vari scenari. In questo caso, vedi i seguenti articoli della Knowledge Base.

* [IIS 6/7: Il metodo POST restituisce 404](https://helpx.adobe.com/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6: Richieste a URL che contengono il percorso di base `/bin` return a `404 Not Found`](https://helpx.adobe.com/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

Controlla anche che la directory principale della cache del Dispatcher e la directory principale del documento IIS siano impostate sulla stessa directory.

## Problemi durante l’eliminazione dei modelli di flussi di lavoro {#problems-deleting-workflow-models}

**Sintomi**

Problemi durante il tentativo di eliminare i modelli di flussi di lavoro quando si accede a un’istanza Autore AEM tramite Dispatcher.

**Passaggi da riprodurre:**

1. Accedi all’istanza di authoring (verifica che le richieste vengano instradate tramite Dispatcher).
1. Creare un flusso di lavoro; ad esempio, con il titolo impostato su workflowToDelete.
1. Verifica che il flusso di lavoro sia stato creato correttamente.
1. Seleziona e fai clic con il pulsante destro del mouse sul flusso di lavoro, quindi fai clic su **Elimina**.

1. Fai clic su **Sì** per confermare.
1. Viene visualizzata una finestra di messaggio di errore che mostra quanto segue:\
   “`ERROR 'Could not delete workflow model!!`”.

**Risoluzione**

Aggiungi le seguenti intestazioni alla sezione `/clientheaders` del file `dispatcher.any`:

* `x-http-method-override`
* `x-requested-with`

```
{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}
```

## Interferenza con mod_dir (Apache) {#interference-with-mod-dir-apache}

Questo processo descrive il modo in cui Dispatcher interagisce con `mod_dir` all’interno del server web Apache, in quanto può causare vari effetti potenzialmente imprevisti:

### Apache 1.3 {#apache}

In Apache 1.3, `mod_dir` gestisce ogni richiesta in cui l’URL viene mappato su una directory nel file system.

Si verifica una delle seguenti alternative:

* La richiesta viene reindirizzata a un file `index.html` esistente
* Viene generato un elenco di directory

Quando Dispatcher è abilitato, elabora tali richieste registrandosi come gestore per il tipo di contenuto `httpd/unix-directory`.

### Apache 2.x {#apache-x}

In Apache 2.x le cose sono diverse. Un modulo può gestire diverse fasi della richiesta, ad esempio la correzione dell’URL. La `mod_dir` gestisce questa fase reindirizzando una richiesta (quando l’URL è mappato su una directory) all’URL con un `/` aggiunto.

Il Dispatcher non intercetta il `mod_dir` correggi, ma gestisce completamente la richiesta all’URL reindirizzato (ovvero con `/` aggiunto). Questo processo potrebbe causare un problema se il server remoto (ad esempio, AEM) gestisce le richieste di `/a_path` in modo diverso rispetto alle richieste a `/a_path/` (quando `/a_path` viene mappato su una directory esistente).

Se si verifica questa situazione, è necessario:

* disable `mod_dir` per `Directory` o `Location` sottostruttura gestita da Dispatcher

* Utilizzare `DirectorySlash Off` per configurare `mod_dir` in modo da evitare che venga aggiunta la `/`
