---
title: Installazione di Dispatcher
seo-title: Installazione di AEM Dispatcher
description: Scopri come installare il modulo Dispatcher su Microsoft Internet Information Server, Apache Web Server e Sun Java Web Server-iPlanet.
seo-description: Scopri come installare il modulo Dispatcher AEM su Microsoft Internet Information Server, Apache Web Server e Sun Java Web Server-iPlanet.
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 35739785aa835a0b995fab4710a0e37bd0ff62b4
workflow-type: tm+mt
source-wordcount: '3684'
ht-degree: 1%

---

# Installazione di Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

Utilizza la pagina [Note sulla versione di Dispatcher](release-notes.md) per ottenere il file di installazione di Dispatcher più recente per il sistema operativo e il server web. I numeri di rilascio del Dispatcher sono indipendenti dai numeri di versione di Adobe Experience Manager e sono compatibili con le versioni Adobe Experience Manager 6.x, 5.x e Adobe CQ 5.x.

>[!NOTE]
>
>Adobe Experience Manager 6.5 richiede Dispatcher versione 4.3.2 o successiva. Tuttavia, le versioni di Dispatcher sono indipendenti da AEM, ad esempio, Dispatcher versione 4.3.2 è compatibile anche con Adobe Experience Manager 6.4.

Viene utilizzata la seguente convenzione per la denominazione dei file:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

Ad esempio, il file `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` contiene Dispatcher versione 4.3.1 per un server web Apache 2.4 che viene eseguito su Linux i686 e viene compilato utilizzando il formato **tar**.

Nella tabella seguente è riportato l’identificatore del server Web utilizzato nei nomi dei file per ogni server Web:

| Server web | Kit di installazione |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;altri parametri> |
| Microsoft Internet Information Server 7.5, 8, 8.5 | dispatcher-**iis**-&lt;altri parametri> |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;altri parametri> |

>[!CAUTION]
>
>Installa la versione più recente di Dispatcher disponibile per la piattaforma. Su base annua, devi aggiornare l’istanza di Dispatcher per utilizzare la versione più recente per sfruttare i miglioramenti apportati al prodotto.

Ogni archivio contiene i file seguenti:

* moduli di Dispatcher
* un file di configurazione di esempio
* il file README contenente le istruzioni di installazione e le informazioni dell&#39;ultimo minuto
* il file CHANGES in cui sono elencati i problemi risolti nelle versioni correnti e precedenti

>[!NOTE]
>
>Prima di avviare l&#39;installazione, controlla il file README per eventuali modifiche dell&#39;ultimo minuto/note specifiche della piattaforma.

<!-- 

Comment Type: draft

<h3>Supported Web Servers</h3>

 -->

<!-- 

Comment Type: draft

<p>The following web servers are supported for use with Dispatcher version 4.1.12:</p>

 -->

<!-- 

Comment Type: draft

<p>The following sections detail the specific web server installation procedures.</p>

 -->

## Microsoft Internet Information Server {#microsoft-internet-information-server}

Per informazioni su come installare questo server web, consulta le risorse seguenti:

* Documentazione di Microsoft su Internet Information Server
* [&quot;Sito ufficiale Microsoft IIS&quot;](https://www.iis.net/)

### Componenti IIS richiesti {#required-iis-components}

Le versioni 8.5 e 10 di IIS richiedono l&#39;installazione dei seguenti componenti IIS:

* Estensioni ISAPI

Inoltre, è necessario aggiungere il ruolo Server web (IIS). Utilizzare Server Manager per aggiungere il ruolo e i componenti.

## Microsoft IIS - Installazione del modulo Dispatcher {#microsoft-iis-installing-the-dispatcher-module}

L&#39;archivio richiesto per Microsoft Internet Information System è:

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

Il file ZIP contiene i seguenti file:

| File | Descrizione |
|--- |--- |
| `disp_iis.dll` | Il file della libreria di collegamenti dinamici di Dispatcher. |
| `disp_iis.ini` | File di configurazione per IIS. Questo esempio può essere aggiornato in base alle tue esigenze. **Nota**: Il file ini deve avere lo stesso nome-radice della dll. |
| `dispatcher.any` | Un file di configurazione di esempio per Dispatcher. |
| `author_dispatcher.any` | Un file di configurazione di esempio per Dispatcher che lavora con l’istanza di authoring. |
| README | File Leggimi contenente istruzioni di installazione e informazioni sull’ultimo minuto. **Nota**: Controllare il file prima di avviare l&#39;installazione. |
| MODIFICHE | Modifica file in cui sono elencati i problemi risolti nelle versioni corrente e passate. |

Per copiare i file Dispatcher nella posizione corretta, attenersi alla procedura descritta di seguito.

1. Utilizzare Esplora risorse per creare la directory `<IIS_INSTALLDIR>/Scripts`, ad esempio `C:\inetpub\Scripts`.

1. Estrai i seguenti file dal pacchetto Dispatcher in questa directory di script:

   * `disp_iis.dll`
   * `disp_iis.ini`
   * Uno dei file seguenti a seconda che Dispatcher stia lavorando con un’istanza di authoring AEM o di pubblicazione:
      * Istanza autore: `author_dispatcher.any`
      * Pubblica istanza: `dispatcher.any`

## Microsoft IIS - Configurare il file INI del Dispatcher {#microsoft-iis-configure-the-dispatcher-ini-file}

Modifica il file `disp_iis.ini` per configurare l’installazione di Dispatcher. Il formato di base del file `.ini` è il seguente:

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

Nella tabella seguente sono descritte le singole proprietà.

| Parametro | Descrizione |
|--- |--- |
| configpath | La posizione di `dispatcher.any` all&#39;interno del file system locale (percorso assoluto). |
| file di registro | Posizione del file `dispatcher.log`. Se non è impostato, i messaggi di log vanno al registro eventi di windows. |
| loglevel | Definisce il livello di registro utilizzato per inviare messaggi al registro eventi. È possibile specificare i seguenti valori:Livello di log per il file di log: <br/>0 - solo messaggi di errore. <br/>1 - errori e avvisi. <br/>2 - errori, avvisi e messaggi informativi  <br/>3 - errori, avvisi, messaggi informativi e di debug. <br/>**Nota**: Si consiglia di impostare il livello di log su 3 durante l&#39;installazione e il test, quindi su 0 quando si esegue in un ambiente di produzione. |
| autorizzazione sostitutiva | Specifica come vengono gestite le intestazioni di autorizzazione nella richiesta HTTP. I seguenti valori sono validi:<br/>0 - Le intestazioni di autorizzazione non vengono modificate. <br/>1 - Sostituisce qualsiasi intestazione denominata &quot;Authorization&quot; diversa da &quot;Basic&quot; con il suo  `Basic <IIS:LOGON\_USER>` equivalente.<br/> |
| variabili server | Definisce il modo in cui vengono elaborate le variabili del server.<br/>0 - Le variabili del server IIS non vengono inviate né a Dispatcher né a AEM. <br/>1 - tutte le variabili del server IIS (come  `LOGON\_USER, QUERY\_STRING, ...`) vengono inviate al Dispatcher, insieme alle intestazioni della richiesta (e anche all&#39;istanza AEM se non è memorizzata nella cache).  <br/>Le variabili del server includono  `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` e molte altre. Vedi la documentazione IIS per l&#39;elenco completo delle variabili, con i dettagli. |
| enable_chunked_transfer | Definisce se abilitare (1) o disabilitare (0) il trasferimento con blocchi per la risposta del client. Il valore predefinito è 0. |

Esempio di configurazione:

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Configurazione di Microsoft IIS {#configuring-microsoft-iis}

Configura IIS per integrare il modulo ISAPI di Dispatcher. In IIS si utilizza la mappatura di applicazioni con caratteri jolly.

### Configurazione di Accesso anonimo - IIS 8.5 e 10 {#configuring-anonymous-access-iis-and}

L’agente di replica Flush predefinito sull’istanza Author è configurato in modo da non inviare credenziali di sicurezza con richieste di scaricamento. Pertanto, il sito web che utilizzi una cache del Dispatcher deve consentire l’accesso anonimo.

Se il sito web utilizza un metodo di autenticazione, l’agente di replica Flush deve essere configurato di conseguenza.

1. Apri IIS Manager e seleziona il sito web che utilizzi come cache del dispatcher.
1. Utilizzando la modalità Visualizzazione funzionalità, nella sezione IIS fare doppio clic su Autenticazione.
1. Se l&#39;autenticazione anonima non è abilitata, selezionare Autenticazione anonima e nell&#39;area Azioni fare clic su Abilita.

### Integrazione del modulo ISAPI di Dispatcher - IIS 8.5 e 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Segui la procedura seguente per aggiungere il modulo ISAPI di Dispatcher a IIS.

1. Apri Gestione IIS.
1. Seleziona il sito web che utilizzi come cache del Dispatcher.
1. Utilizzando la modalità Visualizzazione funzioni, nella sezione IIS fare doppio clic su Mappature gestore.
1. Nel pannello Azioni della pagina Handler Mappings (Mappature gestore), fate clic su Add Wildcard Script Map (Aggiungi mappa script jolly), aggiungete i seguenti valori di proprietà e fate clic su OK:

   * Percorso richiesta: *
   * Eseguibile: Percorso assoluto del file disp_iis.dll, ad esempio `C:\inetpub\Scripts\disp_iis.dll`.
   * Nome: Un nome descrittivo per la mappatura del gestore, ad esempio `Dispatcher`.

1. Nella finestra di dialogo visualizzata, per aggiungere la libreria disp_iis.dll all&#39;elenco delle restrizioni ISAPI e CGI, fare clic su Sì.

   Per IIS 7.0 e 7.5 la configurazione è completa. Se si configura IIS 8.0, procedere con i passaggi successivi.

1. (IIS 8.0) Nell&#39;elenco delle mappature dei gestori, selezionare quella appena creata e nell&#39;area Azioni fare clic su Modifica.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script fare clic sul pulsante Richiedi restrizioni.
1. (IIS 8.0) Per garantire che il gestore sia utilizzato per i file e le cartelle che non sono ancora memorizzati nella cache, deselezionare Invoke Handler Only If Request Is Mapped To, quindi fare clic su OK.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script fare clic su OK.

### Configurazione dell’accesso alla cache - IIS 8.5 e 10 {#configuring-access-to-the-cache-iis-and}

Fornisci all’utente predefinito del pool di app l’accesso in scrittura alla cartella utilizzata come cache del Dispatcher.

1. Fai clic con il pulsante destro del mouse sulla cartella principale del sito web che utilizzi come cache del Dispatcher e fai clic su Proprietà, ad esempio `C:\inetpub\wwwroot`.
1. Nella scheda Protezione fare clic su Modifica, quindi nella finestra di dialogo Autorizzazioni fare clic su Aggiungi. Viene visualizzata una finestra di dialogo per la selezione degli account utente. Fare clic sul pulsante Posizioni, selezionare il nome del computer e quindi fare clic su OK.

   Tieni aperta questa finestra di dialogo durante il completamento del passaggio successivo.

1. in Gestione IIS, selezionare il sito IIS utilizzato per la cache del Dispatcher e fare clic su Impostazioni avanzate sul lato destro della finestra.
1. Selezionare il valore della proprietà Pool di applicazioni e copiarla negli Appunti.
1. Torna alla finestra di dialogo aperta. Nella casella Immettere i nomi degli oggetti da selezionare digitare `IIS AppPool\`, quindi incollare il contenuto degli Appunti. Il valore deve avere un aspetto simile al seguente:

   `IIS AppPool\DefaultAppPool`

1. Fare clic sul pulsante Controlla nomi. Quando Windows risolve l&#39;account utente, fare clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella del dispatcher, seleziona l’account appena aggiunto, abilita tutte le autorizzazioni per l’account **eccetto Controllo completo** e fai clic su OK. Fare clic su OK per chiudere la finestra di dialogo Proprietà cartella.

### Registrazione del tipo di MIME JSON - IIS 8.5 e 10 {#registering-the-json-mime-type-iis-and}

Utilizza la seguente procedura per registrare il tipo MIME JSON quando desideri che Dispatcher consenta le chiamate JSON.

1. In Gestione IIS, seleziona il sito Web e, utilizzando Visualizzazione funzioni, fai doppio clic su Tipi di MIME.
1. Se l&#39;estensione JSON non è presente nell&#39;elenco, nel pannello Azioni fai clic su Aggiungi, immetti i seguenti valori di proprietà, quindi fai clic su OK:

   * Estensione nome file: `.json`
   * Tipo MIME: `application/json`

### Rimozione del segmento nascosto del bin - IIS 8.5 e 10 {#removing-the-bin-hidden-segment-iis-and}

Per rimuovere il segmento nascosto `bin`, attenersi alla procedura descritta di seguito. I siti Web non nuovi possono contenere questo segmento nascosto.

1. In Gestione IIS, selezionare il sito Web e, utilizzando Visualizzazione funzioni, fare doppio clic su Filtro richieste.
1. Selezionare il segmento `bin`, fare clic su Rimuovi e nella finestra di dialogo di conferma fare clic su Sì.

### Registrazione dei messaggi IIS su un file - IIS 8.5 e 10 {#logging-iis-messages-to-a-file-iis-and}

Segui la procedura seguente per scrivere i messaggi di log del Dispatcher in un file di log invece che nel registro eventi di Windows. È necessario configurare Dispatcher per utilizzare il file di registro e fornire a IIS l’accesso in scrittura al file.

1. Utilizzare Esplora risorse per creare una cartella denominata `dispatcher` sotto la cartella dei registri dell&#39;installazione di IIS. Il percorso di questa cartella per un&#39;installazione tipica è `C:\inetpub\logs\dispatcher`.

1. Fai clic con il pulsante destro del mouse sulla cartella del dispatcher e fai clic su Proprietà.
1. Nella scheda Protezione fare clic su Modifica, quindi nella finestra di dialogo Autorizzazioni fare clic su Aggiungi. Viene visualizzata una finestra di dialogo per la selezione degli account utente. Fare clic sul pulsante Posizioni, selezionare il nome del computer e quindi fare clic su OK.

   Tieni aperta questa finestra di dialogo durante il completamento del passaggio successivo.

1. in Gestione IIS, selezionare il sito IIS utilizzato per la cache del Dispatcher e fare clic su Impostazioni avanzate sul lato destro della finestra.
1. Selezionare il valore della proprietà Pool di applicazioni e copiarla negli Appunti.
1. Torna alla finestra di dialogo aperta. Nella casella Immettere i nomi degli oggetti da selezionare digitare `IIS AppPool\`, quindi incollare il contenuto degli Appunti. Il valore deve avere un aspetto simile al seguente:

   `IIS AppPool\DefaultAppPool`

1. Fare clic sul pulsante Controlla nomi. Quando Windows risolve l&#39;account utente, fare clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella del dispatcher, seleziona l’account appena aggiunto, abilita tutte le autorizzazioni per l’account **eccetto Controllo completo,** e fai clic su OK. Fare clic su OK per chiudere la finestra di dialogo Proprietà cartella.
1. Utilizza un editor di testo per aprire il file `disp_iis.ini`.
1. Aggiungi una riga di testo simile all’esempio seguente per configurare il percorso del file di log e quindi salva il file:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Passaggi successivi {#next-steps}

Prima di poter iniziare a utilizzare Dispatcher è necessario conoscere:

* [](dispatcher-configuration.md) ConfigureDispatcher
* [Configura ](page-invalidate.md) AEM per lavorare con Dispatcher.

## Server web Apache {#apache-web-server}

>[!CAUTION]
>
>Le istruzioni per l&#39;installazione in **Windows** e **Unix** sono riportate qui. Presta attenzione durante l&#39;esecuzione dei passaggi.

### Installazione di Apache Web Server {#installing-apache-web-server}

Per informazioni sull&#39;installazione di un server Web Apache, leggere il manuale di installazione [online](https://httpd.apache.org/) o nella distribuzione.

>[!CAUTION]
>
>Se stai creando un binario Apache compilando i file di origine, assicurati di attivare il supporto dei moduli **dinamici**. A tale scopo, puoi utilizzare una qualsiasi delle opzioni **—enable-shared** . Includere almeno il modulo `mod_so`.
>
>Ulteriori informazioni sono disponibili nel manuale di installazione di Apache Web Server.

Vedi anche i [Suggerimenti sulla sicurezza](https://httpd.apache.org/docs/2.4/misc/security_tips.html) e i [Rapporti sulla sicurezza](https://httpd.apache.org/security_report.html) di Apache HTTP Server.

### Server Web Apache - Aggiungi il modulo Dispatcher {#apache-web-server-add-the-dispatcher-module}

Il Dispatcher viene fornito come:

* **Windows**: una libreria di collegamenti dinamici (DLL)
* **Unix**: un oggetto dinamico condiviso (DSO)

I file di archivio di installazione contengono i file seguenti, a seconda che sia stato selezionato Windows o Unix:

| File | Descrizione |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows: Il file della libreria di collegamenti dinamici di Dispatcher. |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | Unix: Il file della libreria di oggetti condivisi di Dispatcher. |
| mod_dispatcher.so | Unix: Un collegamento di esempio. |
| http.conf.disp&lt;x> | Un file di configurazione di esempio per il server Apache. |
| dispatcher.any | Un file di configurazione di esempio per Dispatcher. |
| README | File Leggimi contenente istruzioni di installazione e informazioni sull’ultimo minuto. **Nota**: Controllare il file prima di avviare l&#39;installazione. |
| MODIFICHE | Modifica file in cui sono elencati i problemi risolti nelle versioni corrente e passata. |

Per aggiungere Dispatcher al server Web Apache, effettua le seguenti operazioni:

1. Posiziona il file Dispatcher nella directory appropriata del modulo Apache:

   * **Windows**: Luogo  `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **Unix**: Individua la directory  `<APACHE_ROOT>/libexec` o  `<APACHE_ROOT>/modules`in base all’installazione.\
      Copia `dispatcher-apache<options>.so` in questa directory.\
      Per semplificare la manutenzione a lungo termine, puoi anche creare un collegamento simbolico denominato `mod_dispatcher.so` al Dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Copia il file dispatcher.any nella directory `<APACHE_ROOT>/conf`.

   **Nota:** puoi posizionare il file in una posizione diversa, purché la proprietà DispatcherLog del modulo Dispatcher sia configurata di conseguenza. (Vedi Voci di configurazione specifiche per il Dispatcher di seguito.)

### Server web Apache: configurare le proprietà SELinux {#apache-web-server-configure-selinux-properties}

Se esegui Dispatcher su RedHat Linux Kernel 2.6 con SELinux abilitato, potresti riscontrare messaggi di errore come questo nel file di log del dispatcher.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Questo è probabilmente dovuto a una sicurezza SELinux abilitata. Quindi devi eseguire le seguenti attività:

* Configura il contesto SELinux del file del modulo dispatcher.
* Abilitare gli script e i moduli HTTPD per effettuare connessioni di rete.
* Configura il contesto SELinux del docroot, in cui vengono memorizzati i file memorizzati nella cache.

Immettere i seguenti comandi in una finestra terminale, sostituendo `[path to the dispatcher.so file]` con il percorso del modulo Dispatcher installato in Apache Web Server e *`path to the docroot`* con il percorso in cui si trova il docroot (ad esempio `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Server Web Apache: configurare il server Web Apache per il Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

È necessario configurare Apache Web Server utilizzando `httpd.conf`. Nel kit di installazione di Dispatcher troverai un file di configurazione di esempio denominato `httpd.conf.disp<x>`.

Tali misure sono obbligatorie:

1. Accedi a `<APACHE_ROOT>/conf`.
1. Apri `httpd.conf`per la modifica.
1. È necessario aggiungere le seguenti voci di configurazione, nell&#39;ordine elencato:

   * **** LoadModulper caricare il modulo all’avvio.
   * Voci di configurazione specifiche per il Dispatcher, incluse **DispatcherConfig, DispatcherLog** e **DispatcherLogLevel**.
   * **** SetHandlerto attiva il Dispatcher. **LoadModule**.
   * **** ModMimeUsePathInfoto configura il comportamento di  **mod_mime**.

1. (Facoltativo) È consigliabile modificare il proprietario della directory htdocs:

   * Il server apache viene avviato come radice, anche se i processi figlio iniziano come daemon (a scopo di sicurezza). Il DocumentRoot (`<APACHE_ROOT>/htdocs`) deve appartenere al daemon utente:

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

Nella tabella seguente sono riportati alcuni esempi che possono essere utilizzati; le voci esatte sono in base al server Web Apache specifico:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix (presumibilmente collegamento simbolico) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>Il primo parametro di ogni istruzione deve essere scritto esattamente come negli esempi precedenti.
>
>Per informazioni dettagliate su questo comando, vedere i file di configurazione di esempio forniti e la documentazione di Apache Web Server .

**Voci di configurazione specifiche del dispatcher**

Le voci di configurazione specifiche del Dispatcher vengono inserite dopo la voce LoadModule. Nella tabella seguente è riportato un esempio di configurazione applicabile sia per Unix che per Windows:

**Windows e Unix**

```
...
<fModule disp_apache2.c>
DispatcherConfig conf/dispatcher.any
DispatcherLog logs/dispatcher.log DispatcherLogLevel 3
DispatcherNoServerHeader 0 DispatcherDeclineRoot 0
DispatcherUseProcessedURL 0
DispatcherPassError 0
DispatcherKeepAliveTimeout 60
</IfModule>
...
```

I singoli parametri di configurazione:

| Parametro | Descrizione |
|--- |--- |
| DispatcherConfig | Posizione e nome del file di configurazione del Dispatcher. <br/>Quando questa proprietà si trova nella configurazione del server principale, tutti gli host virtuali ereditano il valore della proprietà. Tuttavia, gli host virtuali possono includere una proprietà DispatcherConfig per ignorare la configurazione del server principale. |
| DispatcherLog | Posizione e nome del file di registro. |
| DispatcherLogLevel | Livello di log per il file di log: <br/>0 - Errori <br/>1 - Avvisi <br/>2 - Informazioni <br/>3 - Debug <br/>**Nota**: Si consiglia di impostare il livello di log su 3 durante l&#39;installazione e il test, quindi su 0 quando si esegue in un ambiente di produzione. |
| DispatcherNoServerHeader | *Questo parametro è obsoleto e non ha più alcun effetto.*<br/><br/> Definisce l&#39;intestazione del server da utilizzare:  <br/><ul><li>undefined o 0 - l&#39;intestazione del server HTTP contiene la versione AEM. </li><li>1 - viene utilizzata l&#39;intestazione del server Apache.</li></ul> |
| DispatcherDeclineRoot | Definisce se rifiutare le richieste alla radice &quot;/&quot;: <br/>**0** - accetta le richieste a / <br/>**1** - le richieste a / non sono gestite dal dispatcher; utilizza mod_alias per la mappatura corretta. |
| DispatcherUseProcessedURL | Definisce se utilizzare URL preelaborati per tutte le ulteriori elaborazioni da parte di Dispatcher: <br/>**0** - utilizza l&#39;URL originale passato al server web. <br/>**1**  - il dispatcher utilizza l’URL già elaborato dai gestori che precedono il dispatcher (es.  `mod_rewrite`) invece dell&#39;URL originale passato al server web.  Ad esempio, l’URL originale o quello elaborato corrisponde ai filtri del Dispatcher. L’URL viene utilizzato anche come base per la struttura del file della cache.   Per informazioni su mod_rewrite, consulta la documentazione del sito web Apache; ad esempio, Apache 2.4. Quando si utilizza mod_rewrite, è consigliabile utilizzare il flag &#39;passthrough | PT&#39; (passare al gestore successivo) per forzare il motore di riscrittura a impostare il campo uri della struttura request_rec interna sul valore del campo del nome del file. |
| DispatcherPassError | Definisce come supportare i codici di errore per la gestione di ErrorDocument: <br/>**0** - Dispatcher esegue lo spooling di tutte le risposte di errore al client. <br/>**1**  - Dispatcher non esegue lo spool di una risposta di errore al client (dove il codice di stato è maggiore o uguale a 400), ma trasmette il codice di stato ad Apache, che ad esempio consente a una direttiva ErrorDocument di elaborare tale codice di stato. <br/>**Intervallo di codice**  - Specifica un intervallo di codici di errore per i quali la risposta viene passata ad Apache. Altri codici di errore vengono passati al client. Ad esempio, la configurazione seguente trasmette al client le risposte per l’errore 412 e tutti gli altri errori vengono passati ad Apache: DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | Specifica il timeout di conservazione, in secondi. A partire dalla versione 4.2.0 di Dispatcher, il valore predefinito per la conservazione è 60. Il valore 0 disattiva keep-alive. |
| DispatcherNoCanonURL | L’impostazione di questo parametro su On passa l’URL non elaborato al backend invece di quello canonicalizzato e sovrascrive le impostazioni di DispatcherUseProcessedURL. Il valore predefinito è Disattivato. <br/>**Nota**: Le regole di filtro nella configurazione di Dispatcher verranno sempre valutate in base all’URL bonificato e non all’URL non elaborato. |




>[!NOTE]
>
>Le voci del percorso sono relative alla directory principale di Apache Web Server.

>[!NOTE]
>
>Le impostazioni predefinite per l&#39;intestazione del server sono:
>
>`ServerTokens Full`
>
>`DispatcherNoServerHeader 0`
>
>Mostra la versione AEM (a fini statistici). Se desideri disattivare la disponibilità di tali informazioni nell’intestazione puoi impostare:
>
>`ServerTokens Prod`
>
>Per ulteriori informazioni, consulta la [Documentazione Apache sulla direttiva ServerToken (ad esempio, per Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html) .

**SetHandler**

Dopo queste voci è necessario aggiungere un&#39;istruzione **SetHandler** al contesto della configurazione ( `<Directory>`, `<Location>`) affinché il Dispatcher possa gestire le richieste in arrivo. L’esempio seguente configura Dispatcher per gestire le richieste per il sito web completo:

**Windows e Unix**

```
...  
<Directory />  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

L’esempio seguente configura il Dispatcher per gestire le richieste per un dominio virtuale:

**Windows**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot _\[cache-path\]_\\docs  
<Directory _\[cache-path\]_\\docs>  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

**Unix**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot /usr/apachecache/docs  
<Directory /usr/apachecache/docs>  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

>[!NOTE]
>
>Il parametro dell&#39;istruzione **SetHandler** deve essere scritto *esattamente come negli esempi precedenti*, in quanto questo è il nome del gestore definito nel modulo.
>
>Per informazioni dettagliate su questo comando, vedere i file di configurazione di esempio forniti e la documentazione di Apache Web Server .

**ModMimeUsePathInfo**

Dopo l&#39;istruzione **SetHandler** è necessario aggiungere anche la definizione **ModMimeUsePathInfo** .

>[!NOTE]
>
>Il parametro `ModMimeUsePathInfo` deve essere utilizzato e configurato solo se utilizzi Dispatcher versione 4.0.9 o successiva.
>
>(Dispatcher versione 4.0.9 è stato rilasciato nel 2011. Se utilizzi una versione precedente, l’aggiornamento a una versione recente di Dispatcher sarebbe appropriato).

Il parametro **ModMimeUsePathInfo** deve essere impostato `On` per tutte le configurazioni Apache:

`ModMimeUsePathInfo On`

Il modulo mod_mime (vedi ad esempio [Apache Module mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) viene utilizzato per assegnare i metadati del contenuto al contenuto selezionato per una risposta HTTP. L’impostazione predefinita indica che quando mod_mime determina il tipo di contenuto, verrà considerata solo la parte dell’URL mappata a un file o a una directory.

Quando `On`, il parametro `ModMimeUsePathInfo` specifica che `mod_mime` deve determinare il tipo di contenuto in base all&#39;URL *complete*; ciò significa che le risorse virtuali avranno le informazioni sulle metriche applicate in base alla loro estensione.

L&#39;esempio seguente attiva **ModMimeUsePathInfo**:

**Windows e Unix**

```
...  
<Directory />  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
ModMimeUsePathInfo On  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

### Abilita il supporto per HTTPS (Unix e Linux) {#enable-support-for-https-unix-and-linux}

Dispatcher utilizza OpenSSL per implementare una comunicazione sicura via HTTP. A partire dalla versione del Dispatcher **4.2.0**, sono supportati OpenSSL 1.0.0 e OpenSSL 1.0.1. Il Dispatcher utilizza OpenSSL 1.0.0 per impostazione predefinita. Per utilizzare OpenSSL 1.0.1, utilizza la procedura seguente per creare collegamenti simbolici in modo che Dispatcher utilizzi le librerie OpenSSL installate.

1. Apri un terminale e cambia la directory corrente nella directory in cui sono installate le librerie OpenSSL, ad esempio:

   ```shell
   cd /usr/lib64
   ```

1. Per creare i collegamenti simbolici, immetti i seguenti comandi:

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
>
>Se utilizzi una versione personalizzata di Apache, assicurati che Apache e Dispatcher siano compilati utilizzando la stessa versione di [OpenSSL](https://www.openssl.org/source/).

### Passaggi successivi {#next-steps-1}

Prima di iniziare a utilizzare Dispatcher, è necessario:

* [](dispatcher-configuration.md) ConfigureDispatcher
* [Configura ](page-invalidate.md) AEM per lavorare con Dispatcher.

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
>
>Le istruzioni per gli ambienti Windows e Unix sono descritte qui.
>
>Presta attenzione quando selezioni quale eseguire.

### Sun Java System Web Server / iPlanet - Installazione del server Web {#sun-java-system-web-server-iplanet-installing-your-web-server}

Per informazioni complete su come installare questi server web, consulta la relativa documentazione:

* Sun Java System Web Server
* Server web iPlanet

### Sun Java System Web Server / iPlanet - Aggiungi il modulo Dispatcher {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Il Dispatcher viene fornito come:

* **Windows**: una libreria di collegamenti dinamici (DLL)
* **Unix**: un oggetto dinamico condiviso (DSO)

I file di archivio di installazione contengono i file seguenti, a seconda che sia stato selezionato Windows o Unix:

| File | Descrizione |
|---|---|
| `disp_ns.dll` | Windows: Il file della libreria di collegamenti dinamici di Dispatcher. |
| `dispatcher.so` | Unix: Il file della libreria di oggetti condivisi di Dispatcher. |
| `dispatcher.so` | Unix: Un collegamento di esempio. |
| `obj.conf.disp` | Un esempio di file di configurazione per il server web iPlanet / Sun Java System. |
| `dispatcher.any` | Un file di configurazione di esempio per Dispatcher. |
| README | File Leggimi contenente istruzioni di installazione e informazioni sull’ultimo minuto. Nota: Controllare il file prima di avviare l&#39;installazione. |
| MODIFICHE | Modifica file in cui sono elencati i problemi risolti nelle versioni corrente e passata. |

Utilizza i seguenti passaggi per aggiungere Dispatcher al server web:

1. Posiziona il file Dispatcher nella directory `plugin` del server web:

### Sun Java System Web Server / iPlanet - Configura per il Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

È necessario configurare il server web utilizzando `obj.conf`. Nel kit di installazione di Dispatcher troverai un file di configurazione di esempio denominato `obj.conf.disp`.

1. Accedi a `<WEBSERVER_ROOT>/config`.
1. Apri `obj.conf`per la modifica.
1. Copia la riga che inizia:\
   `Service fn="dispService"`\
   da `obj.conf.disp` alla sezione di inizializzazione di `obj.conf`.

1. Salva le modifiche.
1. Apri `magnus.conf` per la modifica.
1. Copia le due righe che iniziano:\
   `Init funcs="dispService, dispInit"`\
   e\
   `Init fn="dispInit"`\
   da `obj.conf.disp` alla sezione di inizializzazione di `magnus.conf`.

1. Salva le modifiche.

>[!NOTE]
>
>Le seguenti configurazioni devono essere tutte su una riga e i valori `$(SERVER_ROOT)` e `$(PRODUCT_SUBDIR)` devono essere sostituiti dai rispettivi valori.

**Init**

Nella tabella seguente sono riportati alcuni esempi che possono essere utilizzati; le voci esatte sono in base al tuo server web specifico:

**Windows e Unix**

```
...  
Init funcs="dispService,dispInit" fn="load-modules" shlib="$(SERVER\_ROOT)/plugins/dispatcher.so"  
Init fn="dispInit" config="$(PRODUCT\_SUBDIR)/dispatcher.any" loglevel="1" logfile="$(PRODUCT\_SUBDIR)/logs/dispatcher.log"  
keepalivetimeout="60"  
...
```

dove:

| Parametro | Descrizione |
|--- |--- |
| config | Posizione e nome del file di configurazione `dispatcher.any.` |
| file di registro | Posizione e nome del file di registro. |
| loglevel | Livello di log per quando si scrivono messaggi nel file di log: <br/>**0** Errori <br/>**1** Avvisi <br/>**2** Informazioni <br/>**3** Debug <br/>**Nota:** Si consiglia di impostare il livello di log su 3 durante l&#39;installazione e il test e su 0 quando si esegue in un ambiente di produzione. |
| keepalivetimeout | Specifica il timeout di conservazione, in secondi. A partire dalla versione 4.2.0 di Dispatcher, il valore predefinito per la conservazione è 60. Il valore 0 disattiva keep-alive. |

A seconda delle esigenze è possibile definire Dispatcher come servizio per gli oggetti. Per configurare il Dispatcher per l’intero sito web, modifica l’oggetto predefinito:


**Windows**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)\\dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*\\\*"  
...
```

**Unix**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)/dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*/\*"  
...
```

### Passaggi successivi {#next-steps-2}

Prima di iniziare a utilizzare Dispatcher, è necessario:

* [](dispatcher-configuration.md) ConfigureDispatcher
* [Configura ](page-invalidate.md) AEM per lavorare con Dispatcher.
