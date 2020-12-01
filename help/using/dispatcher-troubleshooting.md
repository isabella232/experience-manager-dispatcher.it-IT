---
title: Risoluzione dei problemi di Dispatcher
seo-title: Risoluzione dei problemi AEM dispatcher
description: Scopri come risolvere i problemi del dispatcher.
seo-description: Scopri come risolvere AEM problemi del dispatcher.
uuid: 9c109a48-d921-4b6e-9626-1158cebc41e7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: a612e745-f1e6-43de-b25a-9adcaadab5cf
translation-type: tm+mt
source-git-commit: 9af0dc22d32f1176b84c28a70b1a4701414d434e
workflow-type: tm+mt
source-wordcount: '553'
ht-degree: 7%

---


# Risoluzione dei problemi di Dispatcher {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM, tuttavia la documentazione del dispatcher è incorporata nella documentazione AEM. Usa sempre la documentazione del dispatcher incorporata nella documentazione per la versione più recente di AEM.
>
>Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

>[!NOTE]
>
>Per ulteriori informazioni, controllare anche la [Knowledge Base del dispatcher](https://helpx.adobe.com/cq/kb/index/dispatcher.html), [Troubleshooting Dispatcher Flushing Issues](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) e le [Domande frequenti sui problemi principali del dispatcher](dispatcher-faq.md).

## Controllare la configurazione di base {#check-the-basic-configuration}

Come sempre, i primi passi sono controllare le nozioni di base:

* [Conferma funzionamento di base](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* Controllate tutti i file di registro per il server Web e il dispatcher. Se necessario, aumentare la `loglevel` utilizzata per il dispatcher [logging](/help/using/dispatcher-configuration.md#logging).

* [Controlla la configurazione](/help/using/dispatcher-configuration.md):

   * Hai più dispatcher?

      * Hai determinato quale Dispatcher gestisce il sito Web/pagina su cui stai indagando?
   * Avete implementato dei filtri?

      * Questi fattori influenzano la questione su cui state indagando?


## Strumenti di diagnostica IIS {#iis-diagnostic-tools}

IIS fornisce vari strumenti di analisi, a seconda della versione effettiva:

* IIS 6 - È possibile scaricare e configurare gli strumenti di diagnostica IIS
* IIS 7 - Traccia completamente integrata

che possono essere utili per monitorare l&#39;attività.

## IIS e 404 non trovati {#iis-and-not-found}

Quando si utilizza IIS è possibile che `404 Not Found` venga restituito in vari scenari. In tal caso, consultate i seguenti articoli della Knowledge Base.

* [IIS 6/7: POST metodo restituisce 404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6: Richieste agli URL che contengono la  `/bin` restituzione del percorso di base  `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

È inoltre necessario verificare che la directory principale della cache del dispatcher e la directory principale del documento IIS siano impostate sulla stessa directory.

## Problemi durante l&#39;eliminazione dei modelli di flussi di lavoro {#problems-deleting-workflow-models}

**Sintomi**

Problemi durante il tentativo di eliminare i modelli di workflow quando si accede a un&#39;istanza di creazione AEM tramite il dispatcher.

**Passaggi per riprodurre:**

1. Effettuate l’accesso all’istanza di creazione (verificate che le richieste siano state instradate tramite il dispatcher).
1. Creare un nuovo flusso di lavoro; ad esempio, con il Titolo impostato su workflowToDelete.
1. Verificare che il flusso di lavoro sia stato creato correttamente.
1. Seleziona e fai clic con il pulsante destro del mouse sul flusso di lavoro, quindi fai clic su **Elimina**.

1. Fate clic su **Sì** per confermare.
1. Viene visualizzata una finestra di messaggio di errore che mostra:\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**Risoluzione**

Aggiungete le seguenti intestazioni alla sezione `/clientheaders` del file `dispatcher.any`:

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

Questo descrive il modo in cui il dispatcher interagisce con `mod_dir` all&#39;interno del server Web Apache, in quanto ciò può causare diversi effetti potenzialmente imprevisti:

### Apache 1.3 {#apache}

In Apache 1.3 `mod_dir` gestisce ogni richiesta in cui l&#39;URL viene mappato su una directory del file system.

Può:

* reindirizzare la richiesta a un file `index.html` esistente
* generare un elenco di directory

Quando il dispatcher è abilitato, elabora tali richieste registrandosi come gestore per il tipo di contenuto `httpd/unix-directory`.

### Apache 2.x {#apache-x}

In Apache 2.x le cose sono diverse. Un modulo può gestire diverse fasi della richiesta, ad esempio la correzione URL. `mod_dir` gestisce questo passaggio reindirizzando una richiesta (quando l’URL viene mappato su una directory) all’URL con un  `/` allegato.

Il dispatcher non intercetta la correzione `mod_dir`, ma gestisce completamente la richiesta all&#39;URL reindirizzato (ad es. con l&#39;aggiunta di `/`). Ciò potrebbe causare un problema se il server remoto (ad es. AEM) gestisce le richieste a `/a_path` in modo diverso rispetto alle richieste a `/a_path/` (quando `/a_path` viene mappato su una directory esistente).

In questo caso è necessario:

* disabilitare `mod_dir` per il sottoalbero `Directory` o `Location` gestito dal dispatcher

* utilizzare `DirectorySlash Off` per configurare `mod_dir` per non aggiungere `/`
