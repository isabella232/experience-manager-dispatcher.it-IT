---
title: Installazione del dispatcher
seo-title: Installazione di AEM Dispatcher
description: Scoprite come installare il modulo Dispatcher su Microsoft Internet Information Server, Apache Web Server e Sun Java Web Server-iPlanet.
seo-description: Scoprite come installare il modulo AEM Dispatcher su Microsoft Internet Information Server, Apache Web Server e Sun Java Web Server-iPlanet.
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: Utente
converted: vero
topic-tags: spedizioniere
content-type: riferimento
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
translation-type: tm+mt
source-git-commit: 6d3ff696780ce55c077a1d14d01efeaebcb8db28

---


# Installazione del dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Potreste essere stati reindirizzati a questa pagina se avete seguito un collegamento alla documentazione del dispatcher incorporata nella documentazione di una versione precedente di AEM.

Utilizzare la pagina [Note](release-notes.md) sulla versione del dispatcher per ottenere il file di installazione più recente per il sistema operativo e il server Web in uso. I numeri di rilascio del dispatcher sono indipendenti dai numeri di rilascio di Adobe Experience Manager e sono compatibili con le versioni di Adobe Experience Manager 6.x, 5.x e Adobe CQ 5.x.

Viene utilizzata la seguente convenzione di denominazione file:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

Ad esempio, il `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` file contiene il Dispatcher versione 4.3.1 per un server Web Apache 2.4 che viene eseguito su Linux i686 e viene compilato utilizzando il formato **tar** .

Nella tabella seguente è riportato l’identificatore del server Web utilizzato nei nomi dei file per ciascun server Web:

| Server web | Kit di installazione |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;altri parametri&gt; |
| Microsoft Internet Information Server 7.5, 8, 8.5 | dispatcher-**iis**-&lt;altri parametri&gt; |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;altri parametri&gt; |

>[!NOTE]
>
>Installa la versione più recente del dispatcher disponibile per la piattaforma in uso. Su base annua, è necessario aggiornare l'istanza Dispatcher per utilizzare la versione più recente per sfruttare i miglioramenti apportati al prodotto.

Ogni archivio contiene i file seguenti:

* moduli Dispatcher
* un file di configurazione di esempio
* il file README che contiene le istruzioni di installazione e le informazioni dell'ultimo minuto
* il file CHANGES in cui sono elencati i problemi risolti nelle release correnti e passate

>[!NOTE]
>
>Prima di avviare l'installazione, controllare il file README per eventuali modifiche dell'ultimo minuto/note specifiche della piattaforma.

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

Per informazioni su come installare questo server Web, consultate le risorse seguenti:

* Documentazione di Microsoft su Internet Information Server
* ["Il sito ufficiale Microsoft IIS"](https://www.iis.net/)

### Componenti IIS richiesti {#required-iis-components}

Le versioni di IIS 8.5 e 10 richiedono l'installazione dei seguenti componenti IIS:

* Estensioni ISAPI

È inoltre necessario aggiungere il ruolo Server Web (IIS). Utilizzare Server Manager per aggiungere ruolo e componenti.

## Microsoft IIS - Installazione del modulo Dispatcher {#microsoft-iis-installing-the-dispatcher-module}

L'archivio richiesto per Microsoft Internet Information System è:

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

Il file ZIP contiene i file seguenti:

| File | Descrizione |
|--- |--- |
| `disp_iis.dll` |  Il file della libreria di collegamenti dinamici del dispatcher. |
| `disp_iis.ini` | File di configurazione per IIS. Questo esempio può essere aggiornato in base alle tue esigenze. **Nota**: Il file ini deve avere lo stesso nome-radice della dll. |
| `dispatcher.any` | Un file di configurazione di esempio per il dispatcher. |
| `author_dispatcher.any` | Un file di configurazione di esempio per il dispatcher che utilizza l’istanza di creazione. |
| README | File Leggimi che contiene le istruzioni di installazione e le informazioni dell'ultimo minuto. **Nota**: Controllare il file prima di avviare l'installazione. |
| MODIFICHE | Modifica il file in cui sono elencati i problemi risolti nelle release corrente e passate. |

Per copiare i file del dispatcher nella posizione corretta, attenersi alla procedura descritta di seguito.

1. Utilizzare Esplora risorse per creare la `<IIS_INSTALLDIR>/Scripts` directory, ad esempio `C:\inetpub\Scripts`.

1. Estrai i seguenti file dal pacchetto Dispatcher in questa directory di script:

   * `disp_iis.dll`
   * `disp_iis.ini`
   * A seconda se Dispatcher utilizza un’istanza di creazione o di pubblicazione AEM:
      * Istanza autore: `author_dispatcher.any`
      * Pubblica istanza: `dispatcher.any`

## Microsoft IIS - Configurare il file INI del dispatcher {#microsoft-iis-configure-the-dispatcher-ini-file}

Modificate il `disp_iis.ini` file per configurare l'installazione del dispatcher. Il formato di base del `.ini` file è il seguente:

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
| configpath | La posizione all' `dispatcher.any` interno del file system locale (percorso assoluto). |
| logfile | Posizione del `dispatcher.log` file. Se non viene impostato, i messaggi di registro vanno al registro eventi di Windows. |
| loglevel | Definisce il livello di registro utilizzato per inviare messaggi al registro eventi. È possibile specificare i seguenti valori:Livello di registro per il file di registro: <br/>0 - solo messaggi di errore. <br/>1 - errori e avvertenze. <br/>2 - errori, avvisi e messaggi informativi <br/>3 - errori, avvisi, messaggi informativi e di debug. <br/>**Nota**: Si consiglia di impostare il livello di registro su 3 durante l'installazione e il test, quindi su 0 quando viene eseguito in un ambiente di produzione. |
| sostituzione | Specifica come vengono gestite le intestazioni di autorizzazione nella richiesta HTTP. I seguenti valori sono validi:<br/>0 - Le intestazioni di autorizzazione non vengono modificate. <br/>1 - Sostituisce con l' `Basic <IIS:LOGON\_USER>` equivalente qualsiasi intestazione denominata "Autorizzazione" diversa da "Base".<br/> |
| servervariables | Definisce la modalità di elaborazione delle variabili del server.<br/>0 - Le variabili del server IIS non vengono inviate né al dispatcher né ad AEM. <br/>1 - tutte le variabili del server IIS (come `LOGON\_USER, QUERY\_STRING, ...`) vengono inviate al dispatcher, insieme alle intestazioni delle richieste (e anche all'istanza di AEM se non è memorizzata nella cache).  <br/>Le variabili del server includono `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` e molte altre. Consulta la documentazione IIS per l'elenco completo delle variabili, con i dettagli. |
| enable_chunked_transfer | Definisce se abilitare (1) o disabilitare (0) il trasferimento con blocco per la risposta del client. Il valore predefinito è 0. |

Esempio di configurazione:

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Configurazione di Microsoft IIS {#configuring-microsoft-iis}

Configurare IIS per integrare il modulo ISAPI del dispatcher. In IIS si utilizza la mappatura dell'applicazione con caratteri jolly.

### Configurazione dell'accesso anonimo - IIS 8.5 e 10 {#configuring-anonymous-access-iis-and}

L'agente di replica Flush predefinito nell'istanza Author è configurato in modo da non inviare credenziali di protezione con richieste di eliminazione. Pertanto, il sito Web che si sta utilizzando una cache del Dispatcher deve consentire l'accesso anonimo.

Se il sito Web utilizza un metodo di autenticazione, l'agente di replica di Flush deve essere configurato di conseguenza.

1. Aprire Gestione IIS e selezionare il sito Web che si sta utilizzando come cache del dispatcher.
1. Utilizzando la modalità Visualizzazione funzioni, nella sezione IIS fare doppio clic su Autenticazione.
1. Se Autenticazione anonima non è abilitata, selezionare Autenticazione anonima, quindi fare clic su Abilita nell'area Azioni.

### Integrazione del modulo ISAPI del dispatcher - IIS 8.5 e 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Utilizzate la procedura seguente per aggiungere il modulo ISAPI del dispatcher a IIS.

1. Aprire Gestione IIS.
1. Selezionate il sito Web che state utilizzando come cache del dispatcher.
1. Utilizzando la modalità Visualizzazione funzioni, nella sezione IIS fare doppio clic su Mappature gestore.
1. Nel pannello Azioni della pagina Mappature gestore, fate clic su Aggiungi mappa script carattere jolly, aggiungete i seguenti valori di proprietà e fate clic su OK:

   * Percorso richiesta: *
   * Eseguibile: Il percorso assoluto del file disp_iis.dll, ad esempio `C:\inetpub\Scripts\disp_iis.dll`.
   * Nome: Un nome descrittivo per la mappatura del gestore, ad esempio `Dispatcher`.

1. Nella finestra di dialogo visualizzata, per aggiungere la libreria disp_iis.dll all'elenco delle limitazioni ISAPI e CGI, fare clic su Sì.

   Per IIS 7.0 e 7.5 la configurazione è completa. Se state configurando IIS 8.0, continuate con i passaggi successivi.

1. (IIS 8.0) Nell'elenco delle mappature dei gestori, selezionare quella appena creata, quindi fare clic su Modifica nell'area Azioni.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script, fare clic sul pulsante Richiedi limitazioni.
1. (IIS 8.0) Per garantire che il gestore sia utilizzato per i file e le cartelle che non sono ancora memorizzati nella cache, deselezionare Richiama gestore solo se la richiesta è mappata su, quindi fare clic su OK.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script, fare clic su OK.

### Configurazione dell'accesso alla cache - IIS 8.5 e 10 {#configuring-access-to-the-cache-iis-and}

Fornite all'utente predefinito del pool di app l'accesso in scrittura alla cartella utilizzata come cache del dispatcher.

1. Fare clic con il pulsante destro del mouse sulla cartella principale del sito Web che si sta utilizzando come cache del Dispatcher e scegliere Proprietà, ad esempio `C:\inetpub\wwwroot`.
1. Nella scheda Protezione fare clic su Modifica, quindi nella finestra di dialogo Autorizzazioni fare clic su Aggiungi. Viene visualizzata una finestra di dialogo per la selezione degli account utente. Fare clic sul pulsante Posizioni, selezionare il nome del computer, quindi fare clic su OK.

   Tenere aperta questa finestra di dialogo mentre si completa il passaggio successivo.

1. in Gestione IIS, selezionare il sito IIS utilizzato per la cache del dispatcher e fare clic su Impostazioni avanzate sul lato destro della finestra.
1. Selezionate il valore della proprietà Pool di applicazioni e copiatelo negli Appunti.
1. Tornate alla finestra di dialogo aperta. Nella casella Immettere i nomi degli oggetti da selezionare, digitare `IIS AppPool\` e incollare il contenuto degli Appunti. Il valore deve essere simile al seguente esempio:

   `IIS AppPool\DefaultAppPool`

1. Fate clic sul pulsante Controlla nomi. Quando Windows risolve l'account utente, fate clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella del dispatcher, selezionate l’account appena aggiunto, abilitate tutte le autorizzazioni per l’account **tranne Controllo** completo e fate clic su OK. Fare clic su OK per chiudere la finestra di dialogo Proprietà cartella.

### Registrazione del tipo JSON Mime - IIS 8.5 e 10 {#registering-the-json-mime-type-iis-and}

Per registrare il tipo MIME JSON, utilizzate la seguente procedura quando desiderate che il Dispatcher consenta le chiamate JSON.

1. In Gestione IIS, selezionate il sito Web e fate doppio clic su Tipi mime utilizzando Visualizzazione funzioni.
1. Se l’estensione JSON non è presente nell’elenco, nel pannello Azioni fate clic su Aggiungi, immettete i seguenti valori di proprietà, quindi fate clic su OK:

   * Estensione nome file: `.json`
   * Tipo MIME: `application/json`

### Rimozione del segmento nascosto del bin - IIS 8.5 e 10 {#removing-the-bin-hidden-segment-iis-and}

Per rimuovere il segmento `bin` nascosto, attenersi alla procedura descritta di seguito. I siti Web non nuovi possono contenere questo segmento nascosto.

1. In Gestione IIS, selezionate il sito Web e fate doppio clic su Richiedi filtro utilizzando Visualizzazione funzioni.
1. Selezionare il `bin` segmento, fare clic su Rimuovi, quindi nella finestra di dialogo di conferma fare clic su Sì.

### Registrazione dei messaggi IIS in un file - IIS 8.5 e 10 {#logging-iis-messages-to-a-file-iis-and}

Utilizzare la procedura seguente per scrivere i messaggi di registro del dispatcher in un file di registro anziché nel registro eventi di Windows. È necessario configurare il dispatcher per l'utilizzo del file di registro e fornire a IIS l'accesso in scrittura al file.

1. Utilizzare Esplora risorse per creare una cartella denominata `dispatcher` sotto la cartella dei file di registro dell'installazione di IIS. Il percorso di questa cartella per un’installazione tipica è `C:\inetpub\logs\dispatcher`.

1. Fare clic con il pulsante destro del mouse sulla cartella del dispatcher e scegliere Proprietà.
1. Nella scheda Protezione fare clic su Modifica, quindi nella finestra di dialogo Autorizzazioni fare clic su Aggiungi. Viene visualizzata una finestra di dialogo per la selezione degli account utente. Fare clic sul pulsante Posizioni, selezionare il nome del computer, quindi fare clic su OK.

   Tenere aperta questa finestra di dialogo mentre si completa il passaggio successivo.

1. in Gestione IIS selezionare il sito IIS utilizzato per la cache del dispatcher, quindi fare clic su Impostazioni avanzate sul lato destro della finestra.
1. Selezionate il valore della proprietà Pool di applicazioni e copiatelo negli Appunti.
1. Tornate alla finestra di dialogo aperta. Nella casella Immettere i nomi degli oggetti da selezionare, digitare `IIS AppPool\` e incollare il contenuto degli Appunti. Il valore deve essere simile al seguente esempio:

   `IIS AppPool\DefaultAppPool`

1. Fate clic sul pulsante Controlla nomi. Quando Windows risolve l'account utente, fate clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella del dispatcher, selezionate l'account appena aggiunto, abilitate tutte le autorizzazioni per l'account** eccetto Controllo completo,** e fate clic su OK. Fare clic su OK per chiudere la finestra di dialogo Proprietà cartella.
1. Usate un editor di testo per aprire il `disp_iis.ini` file.
1. Aggiungete una riga di testo simile al seguente esempio per configurare il percorso del file di registro e quindi salvare il file:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Passaggi successivi {#next-steps}

Prima di iniziare a utilizzare il dispatcher è necessario conoscere:

* [Configura](dispatcher-configuration.md) dispatcher
* [Configurate AEM](page-invalidate.md) per l’utilizzo con Dispatcher.

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>Le istruzioni per l'installazione in **Windows** e **Unix** sono riportate qui. Presta attenzione quando esegui i passaggi.

### Installazione di Apache Web Server {#installing-apache-web-server}

Per informazioni sull'installazione di un server Web Apache, consultare il manuale di installazione [online](https://httpd.apache.org/) o nella distribuzione.

>[!CAUTION]
>
>Se si sta creando un binario Apache compilando i file sorgente, assicurarsi di attivare il supporto **dei moduli** dinamici. Questo può essere fatto utilizzando una qualsiasi delle opzioni **—enable-shared** . Includete almeno il `mod_so` modulo.
>
>Ulteriori informazioni sono disponibili nel manuale di installazione di Apache Web Server.

Consultate anche i suggerimenti [sulla](https://httpd.apache.org/docs/2.4/misc/security_tips.html) sicurezza dei server Apache HTTP e i rapporti [sulla](https://httpd.apache.org/security_report.html)sicurezza.

### Server Web Apache - Aggiungi il modulo Dispatcher {#apache-web-server-add-the-dispatcher-module}

Il dispatcher viene fornito come:

* **Windows**: una libreria di collegamenti dinamici (DLL)
* **Unix**: un oggetto condiviso dinamico (DSO)

I file di archivio dell'installazione contengono i file seguenti, a seconda che sia stato selezionato Windows o Unix:

| File | Descrizione |
|--- |--- |
| disp_apache&lt;x.y&gt;.dll | Windows: Il file della libreria di collegamenti dinamici del dispatcher. |
| dispatcher-apache&lt;x.y&gt;-&lt;rel-nr&gt;.so | Unix: Il file di libreria oggetto condiviso del dispatcher. |
| mod_dispatcher.so | Unix: Un collegamento di esempio. |
| http.conf.disp&lt;x&gt; | Un file di configurazione di esempio per il server Apache. |
| dispatcher.any | Un file di configurazione di esempio per il dispatcher. |
| README | File Leggimi che contiene le istruzioni di installazione e le informazioni dell'ultimo minuto. **Nota**: Controllare il file prima di avviare l'installazione. |
| MODIFICHE | Modifica il file in cui sono elencati i problemi risolti nelle release corrente e passate. |

Per aggiungere Dispatcher al server Web Apache, procedere come segue:

1. Posizionare il file Dispatcher nella directory appropriata del modulo Apache:

   * **Windows**: Posizione `disp_apache<x.y>.dll``<APACHE_ROOT>/modules`
   * **Unix**: Individuate la `<APACHE_ROOT>/libexec` o la `<APACHE_ROOT>/modules`directory in base all'installazione.\
      Copia `dispatcher-apache<options>.so` in questa directory.\
      Per semplificare la manutenzione a lungo termine è inoltre possibile creare un collegamento simbolico denominato `mod_dispatcher.so` al dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Copiate il file dispatcher.any nella `<APACHE_ROOT>/conf` directory.

   **** Nota: È possibile posizionare il file in una posizione diversa, purché la proprietà DispatcherLog del modulo Dispatcher sia configurata di conseguenza. (Vedete Voci di configurazione specifiche del dispatcher di seguito.)

### Server Web Apache - Configurare le proprietà SELLinux {#apache-web-server-configure-selinux-properties}

Se si esegue Dispatcher su RedHat Linux Kernel 2.6 con SELLinux abilitato, è possibile che si verifichino messaggi di errore come questo nel file di registro del dispatcher.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Ciò è probabilmente dovuto a una protezione SELLinux abilitata. A questo punto è necessario eseguire le seguenti operazioni:

* Configurare il contesto SELLinux del file del modulo dispatcher.
* Abilitare gli script e i moduli HTTPD per effettuare connessioni di rete.
* Configurare il contesto SELLinux del docroot, in cui vengono memorizzati i file memorizzati nella cache.

Immettete i seguenti comandi in una finestra terminale, sostituendo `[path to the dispatcher.so file]` con il percorso del modulo Dispatcher installato sul server Web Apache e *`path to the docroot`* con il percorso in cui si trova il docroot (ad esempio `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Server Web Apache - Configurare il server Web Apache per il dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

Il server Web Apache deve essere configurato utilizzando `httpd.conf`. Nel kit di installazione del dispatcher troverete un file di configurazione di esempio denominato `httpd.conf.disp<x>`.

Queste misure sono obbligatorie:

1. Accedi a `<APACHE_ROOT>/conf`.
1. Apri `httpd.conf`per la modifica.
1. È necessario aggiungere le seguenti voci di configurazione, nell'ordine indicato:

   * **LoadModule** per caricare il modulo all'avvio.
   * Voci di configurazione specifiche del dispatcher, inclusi **DispatcherConfig, DispatcherLog** e **DispatcherLogLevel**.
   * **SetHandler** per attivare il dispatcher. **LoadModule**.
   * **ModMimeUsePathInfo** per configurare il comportamento di **mod_mime**.

1. (Facoltativo) È consigliabile modificare il proprietario della directory htdocs:

   * Il server apache viene avviato come radice, anche se i processi secondari iniziano come daemon (a scopo di sicurezza). DocumentRoot (`<APACHE_ROOT>/htdocs`) deve appartenere al demone utente:

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

Nella tabella seguente sono riportati alcuni esempi utilizzabili; le voci esatte sono in base al server Web Apache specifico:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix (supponendo un collegamento simbolico) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>Il primo parametro di ogni istruzione deve essere scritto esattamente come negli esempi precedenti.
>
>Per informazioni dettagliate su questo comando, consultate gli esempi di file di configurazione forniti e la documentazione di Apache Web Server.

**Voci di configurazione specifiche del dispatcher**

Le voci di configurazione specifiche del dispatcher vengono inserite dopo la voce LoadModule. Nella tabella seguente è riportato un esempio di configurazione applicabile sia ad Unix che a Windows:

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
| DispatcherConfig | Posizione e nome del file di configurazione del dispatcher. <br/>Quando questa proprietà si trova nella configurazione del server principale, tutti gli host virtuali ereditano il valore della proprietà. Tuttavia, gli host virtuali possono includere una proprietà DispatcherConfig per ignorare la configurazione del server principale. |
| DispatcherLog | Posizione e nome del file di registro. |
| DispatcherLogLevel | Livello di registro per il file di registro: <br/>0 - Errori <br/>1 - Avvisi <br/>2 - Infos <br/>3 - <br/>**Nota** di debug: Si consiglia di impostare il livello di registro su 3 durante l'installazione e il test, quindi su 0 quando viene eseguito in un ambiente di produzione. |
| DispatcherNoServerHeader | *Questo parametro è obsoleto e non ha più alcun effetto.*<br/><br/> Definisce l’intestazione del server da utilizzare: <br/><ul><li>undefined o 0 - l’intestazione del server HTTP contiene la versione AEM. </li><li>1 - viene utilizzata l'intestazione del server Apache.</li></ul> |
| DispatcherDeclineRoot | Definisce se rifiutare le richieste alla radice "/": <br/>**0** - accettare richieste a / <br/>**1** - le richieste a / non sono gestite dal dispatcher; utilizzate mod_alias per la mappatura corretta. |
| DispatcherUseProcessedURL | Definisce se utilizzare gli URL preelaborati per tutte le ulteriori elaborazioni da parte del dispatcher: <br/>**0** - utilizzate l'URL originale passato al server Web. <br/>**1** - il dispatcher utilizza l'URL già elaborato dai gestori che precedono il dispatcher (ovvero `mod_rewrite`) invece dell’URL originale passato al server Web.  Ad esempio, l’URL originale o quello elaborato corrisponde ai filtri Dispatcher. L'URL viene utilizzato anche come base per la struttura del file della cache.   Per informazioni su mod_rewrite, consultate la documentazione del sito Web Apache; ad esempio, Apache 2.4. Quando si utilizza mod_rewrite, è consigliabile utilizzare il flag 'passthrough | PT' (passare al gestore successivo) per forzare il motore di riscrittura a impostare il campo uri della struttura request_rec interna sul valore del campo del nome del file. |
| DispatcherPassError | Definisce come supportare i codici di errore per la gestione del documento di errore: <br/>**0** - Il dispatcher azzera tutte le risposte di errore al client. <br/>**1** - Il dispatcher non esegue lo spool di una risposta di errore al client (dove il codice di stato è maggiore o uguale a 400), ma trasmette il codice di stato ad Apache, che ad esempio consente a una direttiva ErrorDocument di elaborare tale codice di stato. <br/>**Intervallo** di codici - Specificate un intervallo di codici di errore per il quale la risposta viene passata ad Apache. Altri codici di errore vengono passati al client. Ad esempio, la seguente configurazione trasmette al client le risposte per l'errore 412 e tutti gli altri errori vengono passati ad Apache: DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | Specifica il timeout keep-alive, in secondi. A partire dalla versione 4.2.0 del dispatcher, il valore predefinito keep-alive è 60. Il valore 0 disattiva keep-alive. |
| DispatcherNoCanonURL | Se si imposta questo parametro su On, l’URL non elaborato verrà passato al backend invece di quello canonizzato e verranno ignorate le impostazioni di DispatcherUseProcessedURL. Il valore predefinito è Disattivato. <br/>**Nota**: Le regole del filtro nella configurazione del dispatcher saranno sempre valutate in base all'URL sanzionato, non all'URL non in formato non. |




>[!NOTE]
>
>Le voci del percorso sono relative alla directory principale del server Web Apache.

>[!NOTE]
>
>Le impostazioni predefinite per l'intestazione del server sono: `  
ServerTokens Full` `  
DispatcherNoServerHeader 0`\
che mostra la versione AEM (a fini statistici). Se desiderate disattivare la disponibilità di tali informazioni nell’intestazione potete impostare: `  
ServerTokens Prod`\
Per ulteriori informazioni, consulta la Documentazione [Apache sulla direttiva ServerTokens (ad esempio, per Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html) .

**SetHandler**

Dopo queste voci è necessario aggiungere un'istruzione **SetHandler** al contesto della configurazione ( `<Directory>`, `<Location>`) affinché il dispatcher possa gestire le richieste in entrata. L’esempio seguente configura il dispatcher per gestire le richieste per l’intero sito Web:

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

L’esempio seguente configura il dispatcher per gestire le richieste per un dominio virtuale:

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
Il parametro dell'istruzione **SetHandler** deve essere scritto *esattamente come negli esempi* precedenti, in quanto questo è il nome del gestore definito nel modulo.
Per informazioni dettagliate su questo comando, consultate gli esempi di file di configurazione forniti e la documentazione di Apache Web Server.

**ModMimeUsePathInfo**

Dopo l'istruzione **SetHandler** è necessario aggiungere anche la definizione **ModMimeUsePathInfo** .

>[!NOTE]
Il `ModMimeUsePathInfo` parametro deve essere utilizzato e configurato solo se si utilizza il dispatcher versione 4.0.9 o successiva.
(La versione 4.0.9 del dispatcher è stata rilasciata nel 2011. Se si utilizza una versione precedente, è consigliabile effettuare l’aggiornamento a una versione recente del dispatcher.

Il parametro **ModMimeUsePathInfo** deve essere impostato `On` per tutte le configurazioni Apache:

`ModMimeUsePathInfo On`

Il modulo mod_mime (ad esempio, [Apache Module mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) viene utilizzato per assegnare i metadati del contenuto al contenuto selezionato per una risposta HTTP. L'impostazione predefinita indica che quando mod_mime determina il tipo di contenuto, verrà considerata solo la parte dell'URL mappata a un file o a una directory.

Quando `On`, il `ModMimeUsePathInfo` parametro specifica che `mod_mime` deve determinare il tipo di contenuto in base all'URL *completo* ; ciò significa che alle risorse virtuali verranno applicate delle informazioni sulle metriche in base alla loro estensione.

L’esempio seguente attiva **ModMimeUsePathInfo**:

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

Il dispatcher utilizza OpenSSL per implementare la comunicazione protetta via HTTP. A partire dalla versione **4.2.0** del dispatcher, sono supportati OpenSSL 1.0.0 e OpenSSL 1.0.1. Per impostazione predefinita, il dispatcher utilizza OpenSSL 1.0.0. Per utilizzare OpenSSL 1.0.1, utilizzate la seguente procedura per creare collegamenti simbolici in modo che il dispatcher utilizzi le librerie OpenSSL installate.

1. Aprite un terminale e modificate la directory corrente nella directory in cui sono installate le librerie OpenSSL, ad esempio:

   ```shell
   cd /usr/lib64
   ```

1. Per creare i collegamenti simbolici, immettete i seguenti comandi:

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
Se utilizzate una versione personalizzata di Apache, accertatevi che Apache e Dispatcher siano compilati utilizzando la stessa versione di [OpenSSL](https://www.openssl.org/source/).

### Passaggi successivi {#next-steps-1}

Prima di iniziare a utilizzare il dispatcher è necessario:

* [Configura](dispatcher-configuration.md) dispatcher
* [Configurate AEM](page-invalidate.md) per l’utilizzo con Dispatcher.

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
Le istruzioni per gli ambienti Windows e Unix sono riportate qui.
Fare attenzione quando si seleziona quale eseguire.

### Sun Java System Web Server / iPlanet - Installazione del server Web {#sun-java-system-web-server-iplanet-installing-your-web-server}

Per informazioni complete su come installare questi server Web, consultare la relativa documentazione:

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server / iPlanet - Aggiungi il modulo Dispatcher {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Il dispatcher viene fornito come:

* **Windows**: una libreria di collegamenti dinamici (DLL)
* **Unix**: un oggetto condiviso dinamico (DSO)

I file di archivio dell'installazione contengono i file seguenti, a seconda che sia stato selezionato Windows o Unix:

| File | Descrizione |
|---|---|
| `disp_ns.dll` | Windows: Il file della libreria di collegamenti dinamici del dispatcher. |
| `dispatcher.so` | Unix: Il file di libreria oggetto condiviso del dispatcher. |
| `dispatcher.so` | Unix: Un collegamento di esempio. |
| `obj.conf.disp` | Un file di configurazione di esempio per il server Web iPlanet / Sun Java System. |
| `dispatcher.any` | Un file di configurazione di esempio per il dispatcher. |
| README | File Leggimi che contiene le istruzioni di installazione e le informazioni dell'ultimo minuto. Nota: Controllare il file prima di avviare l'installazione. |
| MODIFICHE | Modifica il file in cui sono elencati i problemi risolti nelle release corrente e passate. |

Per aggiungere il dispatcher al server Web, effettuare le seguenti operazioni:

1. Inserite il file Dispatcher nella `plugin` directory del server Web:

### Sun Java System Web Server / iPlanet - Configurare per il Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

Il server Web deve essere configurato utilizzando `obj.conf`. Nel kit di installazione del dispatcher troverete un file di configurazione di esempio denominato `obj.conf.disp`.

1. Accedi a `<WEBSERVER_ROOT>/config`.
1. Apri `obj.conf`per la modifica.
1. Copiate la linea che inizia:\
   `Service fn="dispService"`\
   dalla `obj.conf.disp` sezione di inizializzazione di `obj.conf`.

1. Salvate le modifiche.
1. Apri `magnus.conf` per la modifica.
1. Copiate le due righe che iniziano:\
   `Init funcs="dispService, dispInit"`\
   e\
   `Init fn="dispInit"`\
   dalla `obj.conf.disp` sezione di inizializzazione di `magnus.conf`.

1. Salvate le modifiche.

>[!NOTE]
Le seguenti configurazioni devono essere tutte su una sola riga e `$(SERVER_ROOT)` `$(PRODUCT_SUBDIR)` devono essere sostituite dai rispettivi valori.

**Init**

Nella tabella seguente sono riportati alcuni esempi utilizzabili; le voci esatte sono in base al server Web specifico:

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
| logfile | Posizione e nome del file di registro. |
| loglevel | <br/> Livello di registro per la scrittura di messaggi nel file di registro: **** 0<br/> Errori **** 1<br/> Avvisi **** 2<br/> Informazioni **** 3<br/> **** Nota di debug: Si consiglia di impostare il livello di registro su 3 durante l'installazione e il test e su 0 quando viene eseguito in un ambiente di produzione. |
| keepalivetimase | Specifica il timeout keep-alive, in secondi. A partire dalla versione 4.2.0 del dispatcher, il valore predefinito keep-alive è 60. Il valore 0 disattiva keep-alive. |

A seconda delle esigenze è possibile definire il dispatcher come un servizio per gli oggetti. Per configurare il dispatcher per l’intero sito Web, modificate l’oggetto predefinito:


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

Prima di iniziare a utilizzare il dispatcher è necessario:

* [Configura](dispatcher-configuration.md) dispatcher
* [Configurate AEM](page-invalidate.md) per l’utilizzo con Dispatcher.
