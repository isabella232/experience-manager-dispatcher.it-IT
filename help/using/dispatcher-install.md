---
title: Installazione del dispatcher
seo-title: Installazione di AEM Dispatcher
description: Scoprite come installare il modulo Dispatcher su Microsoft Internet Information Server, Apache Web Server e Sun Java Web Server-iplanet.
seo-description: Scoprite come installare il modulo di Dispatcher AEM su Microsoft Internet Information Server, Apache Web Server e Sun Java Web Server-iplanet.
uuid: 2384 b 907-1042-4707-b 02 f-fba 2125618 cf
contentOwner: Utente
converted: vero
topic-tags: dispatcher
content-type: riferimento
discoiquuid: f 00 ad 751-6 b 95-4365-8500-e 1 e 0108 d 9536
translation-type: tm+mt
source-git-commit: ec5342f5737f54937493edb0384cdc1d91b13d7c

---


# Installazione del dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporata nella documentazione per una versione precedente di AEM.

Utilizzate la [pagina Note](release-notes.md) sulla versione di Dispatcher per ottenere il file di installazione più recente del dispatcher per il sistema operativo e il server Web utilizzati. I numeri di rilascio del dispatcher sono indipendenti dai numeri di rilascio di Adobe Experience Manager e sono compatibili con le release Adobe Experience Manager 6. x, 5. x e Adobe CQ 5. x.

Viene utilizzata la seguente convenzione di denominazione file:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

Ad esempio, il `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` file contiene la versione 4.3.1 di Dispatcher per un server Web Apache 2.4 eseguito su Linux i 686 e viene incluso nel pacchetto utilizzando il **formato tar** .

La tabella seguente elenca l&#39;identificatore server Web utilizzato nei nomi dei file per ciascun server Web:

| Server web | Kit di installazione |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt; altri parametri &gt; |
| Apache 2.2 | dispatcher-apache **2.2**-&lt; altri parametri &gt; |
| Microsoft Internet Information Server 7.5, 8, 8.5 | dispatcher-**iis**-&lt; altri parametri &gt; |
| Sun Java Web Server iplanet | dispatcher-**ns**-&lt; other parameters &gt; |

>[!NOTE]
>
>È necessario installare la versione più recente di Dispatcher disponibile per la piattaforma. Su base annua, devi aggiornare l&#39;istanza di Dispatcher per utilizzare la versione più recente per sfruttare i miglioramenti apportati al prodotto.

Ogni archivio contiene i file seguenti:

* Moduli del dispatcher
* un esempio di file di configurazione
* il file README contenente istruzioni di installazione e informazioni sull&#39;ultimo minuto
* il file CHANGES che elenca i problemi risolti nelle versioni correnti e passate

>[!NOTE]
>
>Prima di avviare l&#39;installazione, controllate il file Leggimi per eventuali modifiche apportate all&#39;ultimo minuto/note specifiche sulla piattaforma.

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

* Documentazione di Microsoft sul server Internet Information Server
* [&quot; Il sito ufficiale Microsoft IIS &quot;](https://www.iis.net/)

### Componenti IIS richiesti {#required-iis-components}

Le versioni IIS 8.5 e 10 richiedono che siano installati i seguenti componenti IIS:

* Estensioni Isapi

Dovete inoltre aggiungere il ruolo Server Web (IIS). Utilizzate Gestione server per aggiungere il ruolo e i componenti.

## Microsoft IIS - Installazione del modulo Dispatcher {#microsoft-iis-installing-the-dispatcher-module}

L&#39;archivio richiesto per Microsoft Internet Information System è:

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

Il file ZIP contiene i file seguenti:

| File | Descrizione |
|--- |--- |
| `disp_iis.dll` | File di libreria Dynamic Link del dispatcher. |
| `disp_iis.ini` | File di configurazione per IIS. Questo esempio può essere aggiornato in base alle vostre esigenze. **Nota**: Il file ini deve avere lo stesso nome radice della dll. |
| `dispatcher.any` | Un esempio di file di configurazione per il dispatcher. |
| `author_dispatcher.any` | Un file di configurazione di esempio per il dispatcher che utilizza l&#39;istanza Author. |
| LEGGIMI | File Leggimi contenente istruzioni di installazione e informazioni sull&#39;ultimo minuto. **Nota**: Controllate questo file prima di avviare l&#39;installazione. |
| CHANGES | Modifica il file che elenca i problemi risolti nelle release correnti e passate. |

Utilizzate la seguente procedura per copiare i file di Dispatcher nella posizione corretta.

1. Use Windows Explorer to create the `<IIS_INSTALLDIR>/Scripts` directory, for example, `C:\inetpub\Scripts`.

1. Estrarre i seguenti file dal pacchetto Dispatcher in questa directory Scripts:

   * `disp_iis.dll`
   * `disp_iis.ini`
   * Uno dei seguenti file a seconda se il dispatcher sta lavorando con un&#39;istanza di istanza o pubblicazione AEM:
      * Istanza autore: `author_dispatcher.any`
      * Instance instance: `dispatcher.any`

## Microsoft IIS - Configura il file INI del dispatcher {#microsoft-iis-configure-the-dispatcher-ini-file}

Modificate `disp_iis.ini` il file per configurare l&#39;installazione del dispatcher. Il formato di base `.ini` del file è il seguente:

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

La tabella seguente descrive ciascuna proprietà.

| Parametro | Descrizione |
|--- |--- |
| configpath | La posizione dell `dispatcher.any` &#39;interno del file system locale (percorso assoluto). |
| log, logfile | Posizione del `dispatcher.log` file. Se non viene impostato, i messaggi del registro arrivano al registro eventi di Windows. |
| loglevel | Definisce il livello di registro utilizzato per inviare messaggi al registro eventi. È possibile specificare i seguenti valori: Livello di registro per il file di registro: <br/>0 - solo messaggi di errore. <br/>1 - Errori e avvisi. <br/>2 - Errori, avvisi e messaggi informativi <br/>3 - Errori, avvisi, avvisi informativi e messaggi di debug. <br/>**Nota**: Si consiglia di impostare il livello di registro su 3 durante l&#39;installazione e il test, quindi su 0 quando viene eseguito in un ambiente di produzione. |
| replaceauthorization | Specifica in che modo le intestazioni di autorizzazione nella richiesta HTTP vengono gestite. I valori seguenti sono validi:<br/>0 - Le intestazioni di autorizzazione non vengono modificate. <br/>1 - Sostituisce tutte le intestazioni denominate «Autorizzazione» con il relativo `Basic <IIS:LOGON\_USER>` equivalente.<br/> |
| servervariables | Definisce la modalità di elaborazione delle variabili server.<br/>0 - Le variabili server IIS non vengono inviate né al dispatcher né a AEM. <br/>1 - Tutte le variabili server IIS (ad esempio `LOGON\_USER, QUERY\_STRING, ...`) vengono inviate al dispatcher, insieme alle intestazioni della richiesta (e anche all&#39;istanza AEM se non sono memorizzate nella cache). <br/>Le variabili server includono `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` e molti altri. Consulta la documentazione IIS per l&#39;elenco completo delle variabili, con i dettagli. |
| enable_ chunked_ transfer | Definisce se abilitare (1) o disabilitare (0) il trasferimento senza blocchi per la risposta del client. Il valore predefinito è 0. |

Esempio di configurazione:

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Configurazione di Microsoft IIS {#configuring-microsoft-iis}

Configura IIS per integrare il modulo ISAPI del dispatcher. Nell&#39;IIS viene utilizzata la mappatura dell&#39;applicazione jolly.

### Configurazione dell&#39;accesso anonimo - IIS 8.5 e 10 {#configuring-anonymous-access-iis-and}

L&#39;agente di replica Flush predefinito nell&#39;istanza Author è configurato in modo da non inviare credenziali di protezione con richieste di cancellazione. Pertanto, il sito Web che utilizzi una cache del dispatcher deve consentire l&#39;accesso anonimo.

Se il sito Web utilizza un metodo di autenticazione, l&#39;agente di replica Flush deve essere configurato di conseguenza.

1. Aprite Gestione IS e selezionate il sito Web che state utilizzando come cache degli evasioni.
1. Utilizzando la modalità Visualizzazione funzioni, nella sezione IIS fate doppio clic su Autenticazione.
1. Se l&#39;autenticazione anonima non è abilitata, seleziona Autenticazione anonima e, nell&#39;area Azioni, fai clic su Abilita.

### Integrazione del modulo ISAPI Dispatcher - IIS 8.5 e 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Utilizzate la seguente procedura per aggiungere il modulo Isapi del dispatcher all&#39;IIS.

1. Apri Gestione IIS.
1. Selezionate il sito Web che state utilizzando come cache del dispatcher.
1. Utilizzando la modalità Visualizzazione funzioni, nella sezione IIS fate doppio clic su Mappature gestore.
1. Nel pannello Azioni della pagina Mappature gestore, fate clic su Aggiungi mappa script caratteri jolly, aggiungete i seguenti valori delle proprietà e fate clic su OK:

   * Percorso di richiesta: *
   * Eseguibile: Il percorso assoluto del file disp_ iis. dll, ad esempio `C:\inetpub\Scripts\disp_iis.dll`.
   * Nome: Un nome descrittivo per la mappatura del gestore, ad esempio `Dispatcher`.

1. Nella finestra di dialogo visualizzata, per aggiungere la libreria disp_ iis. dll all&#39;elenco Isapi e CGI Limitazioni, fare clic su Sì.

   Per IIS 7.0 e 7.5 la configurazione è completa. Continuate con i passaggi rimanenti se state configurando IIS 8.0.

1. (IIS 8.0) Nell&#39;elenco delle mappature del gestore, selezionate quella appena creata e nell&#39;area Azioni fate clic su Modifica.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script, fare clic sul pulsante Limitazioni richieste.
1. (IIS 8.0) Per assicurarsi che il gestore sia utilizzato per i file e le cartelle che non sono ancora memorizzati nella cache, deselezionate Invoca gestore solo se la richiesta è mappata su, quindi fate clic su OK.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script, fare clic su OK.

### Configurazione dell&#39;accesso alla cache - IIS 8.5 e 10 {#configuring-access-to-the-cache-iis-and}

Fornite l&#39;utente di App Pool predefinito con accesso in scrittura alla cartella utilizzata come cache del dispatcher.

1. Fate clic con il pulsante destro del mouse sulla cartella principale del sito Web che state utilizzando come cache del dispatcher e fate clic su Proprietà, ad esempio `C:\inetpub\wwwroot`.
1. Nella scheda Protezione, fate clic su Modifica e, nella finestra di dialogo Autorizzazioni, fate clic su Aggiungi. Viene aperta una finestra di dialogo per la selezione degli account utente. Fate clic sul pulsante Posizioni, selezionate il nome del computer, quindi fate clic su OK.

   Tenere aperta questa finestra di dialogo mentre si completa il passaggio successivo.

1. in Gestione IIS, selezionate il sito IIS che state utilizzando per la cache del dispatcher; sul lato destro della finestra fate clic su Impostazioni avanzate.
1. Selezionate il valore della proprietà Pool applicazione e copiatelo negli Appunti.
1. Tornare alla finestra di dialogo di apertura. Nella casella Immettere i nomi oggetto da selezionare, digitare `IIS AppPool\` e quindi incollare il contenuto degli Appunti. Il valore deve essere simile al seguente esempio:

   `IIS AppPool\DefaultAppPool`

1. Fate clic sul pulsante Nomi di controllo. Quando Windows risolve l&#39;account utente, fate clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella dispatcher, selezionate l&#39;account appena aggiunto, abilitate tutte le autorizzazioni per l&#39;account **tranne Controllo completo** e fate clic su OK. Fate clic su OK per chiudere la finestra di dialogo Proprietà cartella.

### Registrazione del tipo MIME Mime - IIS 8.5 e 10 {#registering-the-json-mime-type-iis-and}

Utilizzate la seguente procedura per registrare il tipo MIME JSON quando desiderate che il dispatcher consenta le chiamate JSON.

1. In Gestione IIS, seleziona il tuo sito Web e utilizza Visualizzazione funzioni, quindi fai doppio clic su Tipi Mime.
1. Se l&#39;estensione JSON non è nell&#39;elenco, nel pannello Azioni fate clic su Aggiungi, immettete i seguenti valori delle proprietà, quindi fate clic su OK:

   * Estensione nome file: `.json`
   * Tipo MIME: `application/json`

### Rimozione del segmento nascosto del raccoglitore - IIS 8.5 e 10 {#removing-the-bin-hidden-segment-iis-and}

Utilizzate la seguente procedura per rimuovere il `bin` segmento nascosto. I siti Web che non sono nuovi possono contenere questo segmento nascosto.

1. In Manager IIS, selezionate il sito Web e utilizzate Visualizzazione funzioni, fate doppio clic su Filtro richiesta.
1. Selezionate il `bin` segmento, fate clic su Rimuovi e, nella finestra di dialogo di conferma, fate clic su Sì.

### Logging IIS Messages to a File - IIS 8.5 and 10 {#logging-iis-messages-to-a-file-iis-and}

Utilizzare la procedura seguente per scrivere messaggi di registro del dispatcher in un file di registro anziché nel registro evento di Windows. Devi configurare Dispatcher per utilizzare il file di registro e fornire IIS con accesso in scrittura al file.

1. Utilizzate Esplora risorse per creare una cartella denominata `dispatcher` sotto la cartella dei registri dell&#39;installazione IIS. Percorso di questa cartella per un&#39;installazione tipica `C:\inetpub\logs\dispatcher`.

1. Fai clic con il pulsante destro del mouse sulla cartella dispatcher e fai clic su Proprietà.
1. Nella scheda Protezione, fate clic su Modifica e, nella finestra di dialogo Autorizzazioni, fate clic su Aggiungi. Viene aperta una finestra di dialogo per la selezione degli account utente. Fate clic sul pulsante Posizioni, selezionate il nome del computer, quindi fate clic su OK.

   Tenere aperta questa finestra di dialogo mentre si completa il passaggio successivo.

1. in Gestione IIS, selezionate il sito IIS che state utilizzando per la cache del dispatcher; sul lato destro della finestra fate clic su Impostazioni avanzate.
1. Selezionate il valore della proprietà Pool applicazione e copiatelo negli Appunti.
1. Tornare alla finestra di dialogo di apertura. Nella casella Immettere i nomi oggetto da selezionare, digitare `IIS AppPool\` e quindi incollare il contenuto degli Appunti. Il valore deve essere simile al seguente esempio:

   `IIS AppPool\DefaultAppPool`

1. Fate clic sul pulsante Nomi di controllo. Quando Windows risolve l&#39;account utente, fate clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella dispatcher, selezionate l&#39;account appena aggiunto, abilitate tutte le autorizzazioni per l&#39;account** tranne Controllo completo**, quindi fate clic su OK. Fate clic su OK per chiudere la finestra di dialogo Proprietà cartella.
1. Usate un editor di testo per aprire il `disp_iis.ini` file.
1. Aggiungete una riga di testo simile all&#39;esempio seguente per configurare la posizione del file di registro e salvare il file:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Passaggi successivi {#next-steps}

Prima di iniziare a usare il dispatcher è necessario:

* [Configura](dispatcher-configuration.md) dispatcher
* [Corsivo AEM](page-invalidate.md) per lavorare con Dispatcher.

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>Le istruzioni per l&#39;installazione in **Windows** e **Unix** sono descritte qui. Prestate attenzione durante la procedura.

### Installazione di Apache Web Server {#installing-apache-web-server}

Per informazioni su come installare un Apache Web Server leggere il manuale di installazione, [online](https://httpd.apache.org/) o nella distribuzione.

>[!CAUTION]
>
>Se state creando un file binario Apache compilando i file sorgente, accertatevi di attivare **il supporto per moduli dinamici**. Questa operazione può essere eseguita utilizzando uno dei **—opzioni condivise** . Include almeno `mod_so` il modulo.
>
>Ulteriori informazioni sono disponibili nel manuale di installazione di Apache Web Server.

Consultate anche i suggerimenti sulla sicurezza e [i rapporti](https://httpd.apache.org/docs/2.2/misc/security_tips.html) [](https://httpd.apache.org/security_report.html)sulla sicurezza Apache HTTP.

### Apache Web Server - Aggiunta del modulo Dispatcher {#apache-web-server-add-the-dispatcher-module}

Il dispatcher viene fornito come:

* **Windows**: una libreria Dynamic Link (DLL)
* **Unix**: un oggetto condiviso dinamico (DSO)

I file archivio di installazione contengono i file seguenti, a seconda se sono stati selezionati Windows o Unix:

| File | Descrizione |
|--- |--- |
| disp_ apache &lt; x. y &gt;. dll | Windows: File di libreria Dynamic Link del dispatcher. |
| dispatcher-apache &lt; x. y &gt;-&lt; rel-nr &gt;. so | Unix: Il file della libreria oggetto condiviso di Dispatcher. |
| mod_ dispatcher. so | Unix: Un collegamento di esempio. |
| http. conf. disp &lt; x &gt; | Un esempio di file di configurazione per Apache Server. |
| dispatcher. any | Un esempio di file di configurazione per il dispatcher. |
| LEGGIMI | File Leggimi contenente istruzioni di installazione e informazioni sull&#39;ultimo minuto. **Nota**: Controllate questo file prima di avviare l&#39;installazione. |
| CHANGES | Il file in cui sono stati elencati i problemi è stato modificato nelle versioni correnti e passate. |

Utilizza i passaggi seguenti per aggiungere Dispatcher al server Web Apache:

1. Inserite il file del dispatcher nella directory appropriata del modulo Apache:

   * **Windows**: Inserisci `disp_apache<x.y>.dll``<APACHE_ROOT>/modules`
   * **Unix**: Individuate la `<APACHE_ROOT>/libexec``<APACHE_ROOT>/modules`o la directory in base all&#39;installazione.\
      Copia `dispatcher-apache<options>.so` in questa directory.\
      Per semplificare la manutenzione a lungo termine è inoltre possibile creare un collegamento simbolico denominato `mod_dispatcher.so` Dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Copia il dispatcher. any nella `<APACHE_ROOT>/conf` directory.

   **Nota:** Potete posizionare questo file in un percorso diverso, purché la proprietà dispatcherlog del modulo Dispatcher sia configurata di conseguenza. (Vedi Voci di configurazione specifiche per dispatcher sotto).

### Apache Web Server - Configura proprietà selinux {#apache-web-server-configure-selinux-properties}

Se si esegue Dispatcher su redhat Linux Kernel 2.6 con selinux abilitato, è possibile che venga visualizzato in messaggi di errore come nel logfile di registro del dispatcher.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Ciò è probabilmente dovuto a una protezione selinux abilitata. Quindi, è necessario effettuare le seguenti operazioni:

* Configurate il contesto selinux del file del modulo di dispatcher.
* Abilitare gli script e i moduli HTTPD per effettuare connessioni di rete.
* Configurate il contesto selinux del docroot, dove vengono memorizzati i file memorizzati nella cache.

Immettete i seguenti comandi in una finestra terminale, sostituendo `[path to the dispatcher.so file]` con il percorso del modulo di Dispatcher installato sul server Apache Web e *`path to the docroot`* con il percorso in cui si trova il docroot (ad es. `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - Configura Apache Web Server per Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

The Apache Web Server needs to be configured, using `httpd.conf`. In the Dispatcher installation kit you will find an example configuration file named `httpd.conf.disp<x>`.

Questi passaggi sono obbligatori:

1. Accedi a `<APACHE_ROOT>/conf`.
1. Apri `httpd.conf`per la modifica.
1. Nell&#39;elenco elencato è necessario aggiungere le seguenti voci di configurazione:

   * **Loadmodule** per caricare il modulo all&#39;avvio.
   * Voci di configurazione specifiche del dispatcher, inclusi **dispatcherconfig, dispatcherlog** e **dispatcherloglevel**.
   * **Sethandler** per attivare il dispatcher. **Loadmodule**.
   * **Modmimeusepathinfo** per configurare il comportamento **di mod_ mime**.

1. (Facoltativo) Si consiglia di modificare il proprietario della directory htdocs:

   * Il server apache inizia come radice, ma i processi secondari vengono avviati come daemon (a scopo di sicurezza). Documentroot (`<APACHE_ROOT>/htdocs`) deve appartenere al daemon dell&#39;utente:

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**Loadmodule**

Nella tabella seguente sono riportati alcuni esempi d&#39;uso; le voci esatte sono in base al server Web Apache specifico:

|  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix (presupposto simbolico) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>Il primo parametro di ciascuna istruzione deve essere scritto esattamente come negli esempi precedenti.
>
>Per informazioni dettagliate su questo comando, consultate i file di configurazione forniti e la documentazione Apache Web Server.

**Voci di configurazione specifiche del dispatcher**

Le voci di configurazione specifiche del dispatcher vengono inserite dopo la voce loadmodule. La tabella seguente elenca una configurazione di esempio applicabile sia per Unix che per Windows:

**Windows &amp; Unix**

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
| Dispatcherconfig | Posizione e nome del file di configurazione del dispatcher. <br/>Quando questa proprietà si trova nella configurazione del server principale, tutti gli host virtuali ereditano il valore della proprietà. Tuttavia, gli ospitanti virtuali possono includere una proprietà dispatcherconfig per ignorare la configurazione del server principale. |
| Dispatcherlog | Posizione e nome del file di registro. |
| Dispatcherloglevel | Livello di registro per il file di registro: <br/>0 - Errors <br/>1 - Warnings <br/>2 - Infos <br/>3 - Debug <br/>**Note**: Si consiglia di impostare il livello di registro su 3 durante l&#39;installazione e il test, quindi su 0 quando viene eseguito in un ambiente di produzione. |
| Dispatchernoserverheader | *Questo parametro è obsoleto e non ha più effetto.*<br/><br/> Definisce l&#39;Intestazione server da usare: <br/><ul><li>undefined o 0 - L&#39;intestazione del server HTTP contiene la versione AEM. </li><li>1 - L&#39;intestazione server Apache viene utilizzata.</li></ul> |
| Dispatcherweakeroot | Definisce se rifiutare le richieste alla radice &quot;/&quot;: <br/>**0** - accettazione di richieste a/ <br/>**1** - richieste a/non gestite dal dispatcher; utilizzate mod_ alias per la mappatura corretta. |
| Dispatcheruseprocessedurl | Specifica se utilizzare gli URL pre-elaborati per tutte le ulteriori elaborazioni da parte del dispatcher: <br/>**0** - utilizzate l&#39;URL originale passato al server Web. <br/>**1** - Il dispatcher utilizza l&#39;URL già elaborato dai gestori che precedono il dispatcher ( `mod_rewrite`ovvero) anziché l&#39;URL originale passato al server Web. Ad esempio, l&#39;originale o l&#39;URL elaborato viene associato ai filtri del dispatcher. L&#39;URL viene utilizzato anche come base per la struttura del file della cache. Per informazioni su mod_ rewrite, consultate la documentazione del sito Apache ad esempio Apache 2.2. Quando si utilizza mod_ rewrite, è consigliabile utilizzare il flag «passthrough» | PT &#39;(passa al gestore successivo) per forzare il fatto che il motore di riscrittura imposti il campo uri della struttura request_ rec interna sul valore del campo nome file. |
| Dispatcherpasserror | Definisce come supportare i codici di errore per la gestione errordocument: <br/>**0** - Il dispatcher spool tutte le risposte di errore al client. <br/>**1** - Il dispatcher non esegue lo spool di una risposta di errore al client (dove il codice di stato è maggiore o uguale a 400), ma trasmette il codice di stato ad Apache, ad es. consente a una direttiva errordocument di elaborare tale codice di stato. <br/>**Intervallo** di codice - Specifica una serie di codici di errore per i quali la risposta viene passata ad Apache. Altri codici di errore vengono passati al client. Ad esempio, la seguente configurazione trasmette le risposte per l&#39;errore 412 al client, e tutti gli altri errori vengono passati ad Apache: Dispatcherpasserror 400-411,413-417 |
| Dispatcherkeepalivetimeout | Specifica il timeout &quot;keep-alive&quot;, in secondi. A partire da Dispatcher 4.2.0 il valore predefinito di mantenimento è 60. Un valore pari a 0 disattiva il ciclo di vita. |
| Dispatchernocanonurl | Impostando questo parametro su Attivato, l&#39;URL non elaborato viene trasferito al backend anziché quello canonico e le impostazioni di dispatcheruseprocessedurl ignoreranno. Il valore predefinito è Disattivato. <br/>**Nota**: Le regole di filtro nella configurazione del dispatcher saranno sempre valutate rispetto all&#39;URL sanitized, non all&#39;URL non elaborato. |




>[!NOTE]
>
>Le voci percorso sono relative alla directory principale del server Web Apache.

>[!NOTE]
>
>Le impostazioni predefinite per Intestazione server sono: `  
ServerTokens Full``  
DispatcherNoServerHeader 0`\
che mostra la versione di AEM (a scopi statistici). Per disattivare tali informazioni è disponibile nell&#39;intestazione che è possibile impostare: `  
ServerTokens Prod`\
Per ulteriori informazioni, consultate [la documentazione Apache su servertokens Direttiva (ad esempio, Apache 2.2)](https://httpd.apache.org/docs/2.2/mod/core.html) .

**Sethandler**

Dopo queste voci è necessario aggiungere un&#39;istruzione **sethandler** al contesto della configurazione ( `<Directory>`, `<Location>`) affinché il dispatcher possa gestire le richieste in arrivo. L&#39;esempio seguente configura il dispatcher per gestire le richieste del sito Web completo:

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

L&#39;esempio seguente configura il dispatcher per gestire le richieste di un dominio virtuale:

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
Il parametro dell&#39;istruzione **sethandler** deve essere scritto *esattamente come negli esempi precedenti*, in quanto si tratta del nome del gestore definito nel modulo.
Per informazioni dettagliate su questo comando, consultate i file di configurazione forniti e la documentazione Apache Web Server.

**Modmimeusepathinfo**

Dopo l&#39;istruzione **sethandler** è inoltre necessario aggiungere la **definizione modmimeusepathinfo** .

>[!NOTE]
Il `ModMimeUsePathInfo` parametro deve essere utilizzato e configurato solo se utilizzi la versione di Dispatcher 4.0.9 o successiva.
(Nota: la versione 4.0.9 del dispatcher è stata rilasciata nel 2011. Se utilizzi una versione precedente, è opportuno effettuare l&#39;aggiornamento a una versione di Dispatcher recente.

Il parametro **modmimeusepathinfo** deve essere impostato `On` per tutte le configurazioni Apache:

`ModMimeUsePathInfo On`

Il modulo mod_ mime (vedere ad esempio [Apache Module mod_ mime](https://httpd.apache.org/docs/2.2/mod/mod_mime.html)) viene usato per assegnare metadati di contenuto al contenuto selezionato per una risposta HTTP. La configurazione predefinita significa che quando mod_ mime determina il tipo di contenuto, viene considerata solo la porzione dell&#39;URL che viene mappata su un file o una directory.

Quando `On`, il `ModMimeUsePathInfo` parametro specifica che `mod_mime` è necessario determinare il tipo di contenuto in base all&#39;URL *completo* , Ciò significa che le risorse virtuali avranno metainformation applicate in base alla loro estensione.

L&#39;esempio seguente attiva **modmimeusepathinfo**:

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

### Abilitare il supporto per HTTPS (Unix e Linux) {#enable-support-for-https-unix-and-linux}

Il dispatcher utilizza openssl per implementare comunicazioni sicure attraverso HTTP. A partire dalla versione **4.2.0**, sono supportati openssl 1.0.0 e openssl 1.0.1. Per impostazione predefinita, il dispatcher utilizza openssl 1.0.0. Per utilizzare openssl 1.0.1, utilizzate la seguente procedura per creare collegamenti simbolici in modo che il dispatcher utilizzi le librerie openssl installate.

1. Aprite un terminale e modificate la directory corrente nella directory in cui sono installate le librerie openssl, ad esempio:

   ```shell
   cd /usr/lib64
   ```

1. Per creare i collegamenti simbolici, immettete i seguenti comandi:

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
Se utilizzate una versione personalizzata Apache, accertatevi che Apache e Dispatcher siano compilate utilizzando la stessa versione [di openssl](https://www.openssl.org/source/).

### Passaggi successivi {#next-steps-1}

Prima di iniziare a utilizzare il dispatcher è necessario:

* [Configura](dispatcher-configuration.md) dispatcher
* [Corsivo AEM](page-invalidate.md) per lavorare con Dispatcher.

## Sun Java System Web Server/iplanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
Le istruzioni per gli ambienti Windows e Unix sono descritte qui.
Prestate attenzione quando selezionate quali eseguire.

### Sun Java System Web Server/iplanet - Installazione del server Web {#sun-java-system-web-server-iplanet-installing-your-web-server}

Per informazioni dettagliate su come installare tali server Web, consultate la relativa documentazione:

* Sun Java System Web Server
* Iplanet Web Server

### Sun Java System Web Server/iplanet - Aggiunta del modulo Dispatcher {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Il dispatcher viene fornito come:

* **Windows**: una libreria Dynamic Link (DLL)
* **Unix**: un oggetto condiviso dinamico (DSO)

I file archivio di installazione contengono i file seguenti, a seconda se sono stati selezionati Windows o Unix:

| File | Descrizione |
|---|---|
| `disp_ns.dll` | Windows: File di libreria Dynamic Link del dispatcher. |
| `dispatcher.so` | Unix: Il file della libreria oggetto condiviso di Dispatcher. |
| `dispatcher.so` | Unix: Un collegamento di esempio. |
| `obj.conf.disp` | Un esempio di file di configurazione per il server Web iplanet/Sun Java System. |
| `dispatcher.any` | Un esempio di file di configurazione per il dispatcher. |
| LEGGIMI | File Leggimi contenente istruzioni di installazione e informazioni sull&#39;ultimo minuto. Nota: Controllate questo file prima di avviare l&#39;installazione. |
| CHANGES | Il file in cui sono stati elencati i problemi è stato modificato nelle versioni correnti e passate. |

Utilizza i passaggi seguenti per aggiungere il dispatcher al server Web:

1. Inserite il file del dispatcher nella `plugin` directory del server Web:

### Sun Java System Web Server/iplanet - Configura per il dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

È `obj.conf`necessario configurare il server Web. In the Dispatcher installation kit you will find an example configuration file named `obj.conf.disp`.

1. Accedi a `<WEBSERVER_ROOT>/config`.
1. Apri `obj.conf`per la modifica.
1. Copia la linea che inizia:\
   `Service fn="dispService"`\
   dalla `obj.conf.disp` sezione di inizializzazione di `obj.conf`.

1. Salvate le modifiche.
1. Apri `magnus.conf` per la modifica.
1. Copiare le due righe che iniziano:\
   `Init funcs="dispService, dispInit"`\
   e\
   `Init fn="dispInit"`\
   dalla `obj.conf.disp` sezione di inizializzazione di `magnus.conf`.

1. Salvate le modifiche.

>[!NOTE]
Le seguenti configurazioni devono essere tutte su una riga e quelle che `$(SERVER_ROOT)` devono `$(PRODUCT_SUBDIR)` essere sostituite dai rispettivi valori.

**Init**

Nella tabella seguente sono riportati alcuni esempi d&#39;uso; le voci esatte sono in base al server Web specifico:

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
| log, logfile | Posizione e nome del file di registro. |
| loglevel | Livello di registro per la scrittura di messaggi nel file di registro: <br/>**0** Errors <br/>**1** Warnings <br/>**2** Infos <br/>**3** Debug <br/>**Note:** Si consiglia di impostare il livello di registro su 3 durante l&#39;installazione e il test e su 0 quando viene eseguito in un ambiente di produzione. |
| keepalivetimeout | Specifica il timeout &quot;keep-alive&quot;, in secondi. A partire da Dispatcher 4.2.0 il valore predefinito di mantenimento è 60. Un valore pari a 0 disattiva il ciclo di vita. |

A seconda delle esigenze, è possibile definire il dispatcher come servizio per gli oggetti. Per configurare il dispatcher per l&#39;intero sito Web, modificare l&#39;oggetto predefinito:


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
* [Corsivo AEM](page-invalidate.md) per lavorare con Dispatcher.
