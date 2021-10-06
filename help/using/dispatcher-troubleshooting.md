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
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '543'
ht-degree: 100%

---

# Risoluzione dei problemi di Dispatcher {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM, tuttavia la documentazione di Dispatcher è incorporata nella documentazione di AEM. Utilizza sempre la documentazione di Dispatcher incorporata nella documentazione della più recente versione di AEM.
>
>Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

>[!NOTE]
>
>Per ulteriori informazioni, controlla anche la [Knowledge Base di Dispatcher](https://helpx.adobe.com/it/experience-manager/kb/index/dispatcher.html), la [Risoluzione dei problemi di flushingdi Dispatcher](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) e le [Domande frequenti sui problemi principali di Dispatcher](dispatcher-faq.md).

## Controlla la configurazione di base {#check-the-basic-configuration}

Come sempre, i primi passaggi consistono nel controllare la configurazione di base:

* [Verifica il funzionamento di base](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* Controlla tutti i file di registro del server web e di Dispatcher. Se necessario, aumenta il `loglevel` utilizzato per [la registrazione](/help/using/dispatcher-configuration.md#logging) di Dispatcher.

* [Verifica la configurazione](/help/using/dispatcher-configuration.md):

   * Hai più Dispatcher?

      * Hai stabilito quale Dispatcher gestisce il Sito Web/la pagina che stai esaminando?
   * Hai implementato i filtri?

      * I filtri stanno influenzando il problema che stai esaminando?


## Strumenti di diagnostica IIS {#iis-diagnostic-tools}

IIS fornisce vari strumenti di trace, a seconda della versione effettiva:

* IIS 6 - È possibile scaricare e configurare gli strumenti di diagnostica IIS
* IIS 7 - Il tracciamento è completamente integrato

Questi strumento possono aiutarti a monitorare l’attività.

## Impossibile trovare IIS e 404 {#iis-and-not-found}

Quando si utilizza IIS, è possibile che il messaggio `404 Not Found` venga restituito in vari scenari. In questo caso, vedi i seguenti articoli della Knowledge Base.

* [IIS 6/7: Il metodo POST restituisce 404](https://helpx.adobe.com/it/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6: Richieste a URL contenenti il percorso base `/bin` restituiscono `404 Not Found`](https://helpx.adobe.com/it/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

È inoltre necessario verificare che la directory principale della cache di Dispatcher sia la stessa dei documenti IIS.

## Problemi durante l’eliminazione dei modelli di flussi di lavoro {#problems-deleting-workflow-models}

**Sintomi**

Problemi durante il tentativo di eliminare i modelli di flussi di lavoro quando si accede a un’istanza Autore AEM tramite Dispatcher.

**Passaggi da riprodurre:**

1. Accedi all’istanza Autore (verifica che le richieste vengano instradate tramite Dispatcher).
1. Crea un nuovo flusso di lavoro; ad esempio, con il titolo workflowToDelete.
1. Verifica che il flusso di lavoro sia stato creato correttamente.
1. Seleziona e fai clic con il pulsante destro del mouse sul flusso di lavoro, quindi fai clic su **Elimina**.

1. Fai clic su **Sì** per confermare.
1. Si apre una finestra con il seguente messaggio di errore:\
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

Descrive il modo in cui Dispatcher interagisce con `mod_dir` all’interno del server web Apache, in quanto ciò può avere vari effetti potenzialmente imprevisti:

### Apache 1.3 {#apache}

In Apache 1.3 `mod_dir` gestisce ogni richiesta in cui l’URL viene mappato su una directory nel file system.

Si verifica una delle seguenti alternative:

* La richiesta viene reindirizzata a un file `index.html` esistente
* Viene generato un elenco di directory

Quando il dispatcher è abilitato, elabora queste richieste registrandosi come handler del tipo di contenuto `httpd/unix-directory`.

### Apache 2.x {#apache-x}

In Apache 2.x le cose sono diverse. Un modulo può gestire diverse fasi della richiesta, ad esempio la correzione dell’URL. `mod_dir` gestisce questa fase reindirizzando una richiesta (quando l’URL è mappato su una directory) all’URL con una `/` aggiunta alla fine.

Dispatcher non intercetta la correzione `mod_dir`, ma gestisce completamente la richiesta all’URL reindirizzato (vale a dire, con una `/` aggiunta alla fine). Ciò potrebbe causare un problema, se il server remoto (ad esempio, AEM) gestisce le richieste indirizzate a `/a_path` in modo diverso rispetto alle richieste indirizzate a `/a_path/` (quando `/a_path` viene mappato su una directory esistente).

Se ciò si verifica, occorre effettuare una di queste due azioni:

* disattivare `mod_dir` per il sottoalbero `Directory` o `Location` gestito da Dispatcher

* Utilizzare `DirectorySlash Off` per configurare `mod_dir` in modo da evitare che venga aggiunta la `/`
