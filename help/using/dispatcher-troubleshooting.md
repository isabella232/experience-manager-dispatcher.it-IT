---
title: Risoluzione dei problemi di dispatcher
seo-title: Risoluzione dei problemi relativi a AEM Dispatcher
description: Scopri come risolvere problemi relativi a Dispatcher.
seo-description: Scopri come risolvere problemi relativi a Dispatcher AEM.
uuid: 9 c 109 a 48-d 921-4 b 6 e -9626-1158 cebc 41 e 7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: Utente
products: SG_ EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: riferimento
discoiquuid: a 612 e 745-f 1 e 6-43 de-b 25 a -9 adcaadab 5 cf
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# Risoluzione dei problemi di dispatcher {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM, tuttavia la documentazione di Dispatcher è incorporata nella documentazione di AEM. Utilizza sempre la documentazione di Dispatcher incorporata nella documentazione per la versione più recente di AEM.
>
>Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporata nella documentazione per una versione precedente di AEM.

>[!NOTE]
>
>Per ulteriori informazioni, consulta anche la [sezione relativa alla knowledgebase del dispatcher](https://helpx.adobe.com/cq/kb/index/dispatcher.html), [alla risoluzione dei problemi relativi a problemi di cancellazione del dispatcher](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) e alle [domande frequenti](dispatcher-faq.md) sui problemi principali del dispatcher.

## Verifica configurazione di base {#check-the-basic-configuration}

Come prima cosa, controllare le nozioni di base:

* [Conferma operazione di base](#ConfirmBasicOperation)
* Controllate tutti i file di registro per il server Web e il dispatcher. Se necessario, aumentate l&#39; `loglevel` utilizzo per la [registrazione di dispatcher](#Logging).

* [Controlla la configurazione](#ConfiguringtheDispatcher):

   * Hai più dispatcher?

      * Hai determinato quale dispatcher sta gestendo il sito Web/la pagina che stai indagando?
   * Hai implementato i filtri?

      * Queste influiscono sulla questione che stai indagando?


## Strumenti di diagnostica IIS {#iis-diagnostic-tools}

IIS offre vari strumenti di trace, a seconda della versione effettiva:

* IIS 6: gli strumenti diagnostici IIS possono essere scaricati e configurati
* IIS 7: il ricalco è completamente integrato

Questi possono facilitare il monitoraggio dell&#39;attività.

## IIS e 404 non trovato {#iis-and-not-found}

Quando si utilizza IIS potresti venire `404 Not Found` restituito in vari scenari. In tal caso, consultate i seguenti articoli della Knowledgebase.

* [IIS 6/7: POST method restituisce 404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6: Richieste agli URL che contengono il percorso `/bin` di base `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

Controllate inoltre che la radice della cache del dispatcher e la radice del documento IIS siano impostate sulla stessa directory.

## Problemi di eliminazione dei modelli di flusso di lavoro {#problems-deleting-workflow-models}

**Sintomi**

Problemi durante l&#39;eliminazione dei modelli di flusso di lavoro quando si accede a un&#39;istanza di istanza AEM tramite il dispatcher.

**Passaggi per riprodurre:**

1. Accedi all&#39;istanza di authoring (conferma che le richieste vengono instradate attraverso il dispatcher).
1. Creare un nuovo flusso di lavoro; ad esempio, con il titolo impostato su workflowtodelete.
1. Verificate che il flusso di lavoro sia stato creato correttamente.
1. Selezionate e fate clic con il pulsante destro del mouse sul flusso di lavoro, quindi fate clic su **Elimina**.

1. Fate clic su **Sì** per confermare.
1. Viene visualizzata una finestra di messaggio di errore:\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**Risoluzione**

Aggiungete le seguenti intestazioni alla `/clientheaders` sezione del `dispatcher.any` file:

* `x-http-method-override`
* `x-requested-with`

`{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}`

## Interferenza con mod_ dir (Apache) {#interference-with-mod-dir-apache}

Questo descrive in che modo il dispatcher interagisce `mod_dir` all&#39;interno del webserver Web Apache, in quanto può portare a diversi effetti potenzialmente imprevisti:

### Apache 1.3 {#apache}

In Apache 1.3 `mod_dir` viene gestito ogni richiesta in cui l&#39;URL viene mappato su una directory del file system.

Effettua una delle seguenti operazioni:

* reindirizzare la richiesta a un `index.html` file esistente
* generazione di un elenco di directory

Quando il dispatcher è abilitato, elabora tali richieste registrandosi come gestore per il tipo `httpd/unix-directory`di contenuto.

### Apache 2. x {#apache-x}

Le novità Apache 2. x sono diverse. Un modulo può gestire diverse fasi della richiesta, ad esempio la correzione URL. `mod_dir` gestisce l&#39;area di visualizzazione reindirizzando una richiesta (quando l&#39;URL viene mappato su una directory) all&#39;URL con un `/` suffisso.

Il dispatcher non interrompe `mod_dir` la correzione, ma gestisce completamente la richiesta all&#39;URL reindirizzamento (ad es. con `/` l&#39;aggiunta). Questo potrebbe presentare un problema se il server remoto (ad es. AEM) gestisce le richieste `/a_path` in modo diverso rispetto alle richieste a `/a_path/` (quando `/a_path` viene mappata su una directory esistente).

In tal caso è necessario:

* disable `mod_dir` for the `Directory` or `Location` subtree manage by the dispatcher

* utilizzate `DirectorySlash Off` per `mod_dir` configurare non aggiungere `/`
