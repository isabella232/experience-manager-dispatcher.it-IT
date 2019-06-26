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
source-git-commit: 6d3ff696780ce55c077a1d14d01efeaebcb8db28

---


# Installing Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporata nella documentazione per una versione precedente di AEM.

Use the [Dispatcher Release Notes](release-notes.md) page to obtain the latest Dispatcher installation file for your operating system and web server. I numeri di rilascio del dispatcher sono indipendenti dai numeri di rilascio di Adobe Experience Manager e sono compatibili con le release Adobe Experience Manager 6. x, 5. x e Adobe CQ 5. x.

Viene utilizzata la seguente convenzione di denominazione file:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

For example, the `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` file contains Dispatcher version 4.3.1 for an Apache 2.4 web server that runs on Linux i686 and is packaged using the **tar** format.

La tabella seguente elenca l&#39;identificatore server Web utilizzato nei nomi dei file per ciascun server Web:

| Server web | Kit di installazione |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;other parameters&gt; |
| Microsoft Internet Information Server 7.5, 8, 8.5 | dispatcher-**iis**-&lt;other parameters&gt; |
| Sun Java Web Server iplanet | dispatcher-**ns**-&lt;other parameters&gt; |

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

### Required IIS Components {#required-iis-components}

Le versioni IIS 8.5 e 10 richiedono che siano installati i seguenti componenti IIS:

* Estensioni Isapi

Dovete inoltre aggiungere il ruolo Server Web (IIS). Utilizzate Gestione server per aggiungere il ruolo e i componenti.

## Microsoft IIS - Installing the Dispatcher module {#microsoft-iis-installing-the-dispatcher-module}

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
      * Author instance: `author_dispatcher.any`
      * Publish instance: `dispatcher.any`

## Microsoft IIS - Configure the Dispatcher INI File {#microsoft-iis-configure-the-dispatcher-ini-file}

Edit the `disp_iis.ini` file to configure the Dispatcher installation. The basic format of the `.ini` file is as follows:

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
| configpath | The location of `dispatcher.any` within the local file system (absolute path). |
| log, logfile | The location of the `dispatcher.log` file. Se non viene impostato, i messaggi del registro arrivano al registro eventi di Windows. |
| loglevel | Definisce il livello di registro utilizzato per inviare messaggi al registro eventi. The following values may be specified:Log level for the log file: <br/>0 - error messages only. <br/>1 - Errori e avvisi. <br/>2 - Errori, avvisi e messaggi informativi <br/>3 - Errori, avvisi, avvisi informativi e messaggi di debug. <br/>**Nota**: Si consiglia di impostare il livello di registro su 3 durante l&#39;installazione e il test, quindi su 0 quando viene eseguito in un ambiente di produzione. |
| replaceauthorization | Specifica in che modo le intestazioni di autorizzazione nella richiesta HTTP vengono gestite. The following values are valid:<br/>0 - Authorization headers are not modified. <br/>1 - Sostituisce tutte le intestazioni denominate «Autorizzazione» con il relativo `Basic <IIS:LOGON\_USER>` equivalente.<br/> |
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

### Configuring Microsoft IIS {#configuring-microsoft-iis}

Configura IIS per integrare il modulo ISAPI del dispatcher. Nell&#39;IIS viene utilizzata la mappatura dell&#39;applicazione jolly.

### Configuring Anonymous Access - IIS 8.5 and 10 {#configuring-anonymous-access-iis-and}

L&#39;agente di replica Flush predefinito nell&#39;istanza Author è configurato in modo da non inviare credenziali di protezione con richieste di cancellazione. Pertanto, il sito Web che utilizzi una cache del dispatcher deve consentire l&#39;accesso anonimo.

Se il sito Web utilizza un metodo di autenticazione, l&#39;agente di replica Flush deve essere configurato di conseguenza.

1. Aprite Gestione IS e selezionate il sito Web che state utilizzando come cache degli evasioni.
1. Utilizzando la modalità Visualizzazione funzioni, nella sezione IIS fate doppio clic su Autenticazione.
1. Se l&#39;autenticazione anonima non è abilitata, seleziona Autenticazione anonima e, nell&#39;area Azioni, fai clic su Abilita.

### Integrating the Dispatcher ISAPI Module - IIS 8.5 and 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Utilizzate la seguente procedura per aggiungere il modulo Isapi del dispatcher all&#39;IIS.

1. Apri Gestione IIS.
1. Selezionate il sito Web che state utilizzando come cache del dispatcher.
1. Utilizzando la modalità Visualizzazione funzioni, nella sezione IIS fate doppio clic su Mappature gestore.
1. Nel pannello Azioni della pagina Mappature gestore, fate clic su Aggiungi mappa script caratteri jolly, aggiungete i seguenti valori delle proprietà e fate clic su OK:

   * Percorso di richiesta: *
   * Executable: The absolute path of the disp_iis.dll file, for example `C:\inetpub\Scripts\disp_iis.dll`.
   * Name: A descriptive name for the handler mapping, for example `Dispatcher`.

1. Nella finestra di dialogo visualizzata, per aggiungere la libreria disp_ iis. dll all&#39;elenco Isapi e CGI Limitazioni, fare clic su Sì.

   Per IIS 7.0 e 7.5 la configurazione è completa. Continuate con i passaggi rimanenti se state configurando IIS 8.0.

1. (IIS 8.0) Nell&#39;elenco delle mappature del gestore, selezionate quella appena creata e nell&#39;area Azioni fate clic su Modifica.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script, fare clic sul pulsante Limitazioni richieste.
1. (IIS 8.0) Per assicurarsi che il gestore sia utilizzato per i file e le cartelle che non sono ancora memorizzati nella cache, deselezionate Invoca gestore solo se la richiesta è mappata su, quindi fate clic su OK.
1. (IIS 8.0) Nella finestra di dialogo Modifica mappa script, fare clic su OK.

### Configuring Access to the Cache - IIS 8.5 and 10 {#configuring-access-to-the-cache-iis-and}

Fornite l&#39;utente di App Pool predefinito con accesso in scrittura alla cartella utilizzata come cache del dispatcher.

1. Right-click the root folder of the website that you are using as the Dispatcher cache and click Properties, such as `C:\inetpub\wwwroot`.
1. Nella scheda Protezione, fate clic su Modifica e, nella finestra di dialogo Autorizzazioni, fate clic su Aggiungi. Viene aperta una finestra di dialogo per la selezione degli account utente. Fate clic sul pulsante Posizioni, selezionate il nome del computer, quindi fate clic su OK.

   Tenere aperta questa finestra di dialogo mentre si completa il passaggio successivo.

1. in Gestione IIS, selezionate il sito IIS che state utilizzando per la cache del dispatcher; sul lato destro della finestra fate clic su Impostazioni avanzate.
1. Selezionate il valore della proprietà Pool applicazione e copiatelo negli Appunti.
1. Tornare alla finestra di dialogo di apertura. In the Enter The Object Names To Select box, type `IIS AppPool\` and then paste the contents of your clipboard. Il valore deve essere simile al seguente esempio:

   `IIS AppPool\DefaultAppPool`

1. Fate clic sul pulsante Nomi di controllo. Quando Windows risolve l&#39;account utente, fate clic su OK.
1. In the Permissions dialog box for the dispatcher folder, select the account that you just added, enable all of the permissions for the account  **except for Full Control** and click OK. Fate clic su OK per chiudere la finestra di dialogo Proprietà cartella.

### Registering the JSON Mime Type - IIS 8.5 and 10 {#registering-the-json-mime-type-iis-and}

Utilizzate la seguente procedura per registrare il tipo MIME JSON quando desiderate che il dispatcher consenta le chiamate JSON.

1. In Gestione IIS, seleziona il tuo sito Web e utilizza Visualizzazione funzioni, quindi fai doppio clic su Tipi Mime.
1. Se l&#39;estensione JSON non è nell&#39;elenco, nel pannello Azioni fate clic su Aggiungi, immettete i seguenti valori delle proprietà, quindi fate clic su OK:

   * File Name Extension: `.json`
   * MIME Type: `application/json`

### Removing the bin Hidden Segment - IIS 8.5 and 10 {#removing-the-bin-hidden-segment-iis-and}

Use the following procedure to remove the `bin` hidden segment. I siti Web che non sono nuovi possono contenere questo segmento nascosto.

1. In Manager IIS, selezionate il sito Web e utilizzate Visualizzazione funzioni, fate doppio clic su Filtro richiesta.
1. Select the `bin` segment, click Remove, and in the confirmation dialog box click Yes.

### Logging IIS Messages to a File - IIS 8.5 and 10 {#logging-iis-messages-to-a-file-iis-and}

Utilizzare la procedura seguente per scrivere messaggi di registro del dispatcher in un file di registro anziché nel registro evento di Windows. Devi configurare Dispatcher per utilizzare il file di registro e fornire IIS con accesso in scrittura al file.

1. Use Windows Explorer to create a folder named `dispatcher` below the logs folder of the IIS installation. The path of this folder for a typical installation is `C:\inetpub\logs\dispatcher`.

1. Fai clic con il pulsante destro del mouse sulla cartella dispatcher e fai clic su Proprietà.
1. Nella scheda Protezione, fate clic su Modifica e, nella finestra di dialogo Autorizzazioni, fate clic su Aggiungi. Viene aperta una finestra di dialogo per la selezione degli account utente. Fate clic sul pulsante Posizioni, selezionate il nome del computer, quindi fate clic su OK.

   Tenere aperta questa finestra di dialogo mentre si completa il passaggio successivo.

1. in Gestione IIS, selezionate il sito IIS che state utilizzando per la cache del dispatcher; sul lato destro della finestra fate clic su Impostazioni avanzate.
1. Selezionate il valore della proprietà Pool applicazione e copiatelo negli Appunti.
1. Tornare alla finestra di dialogo di apertura. In the Enter The Object Names To Select box, type `IIS AppPool\` and then paste the contents of your clipboard. Il valore deve essere simile al seguente esempio:

   `IIS AppPool\DefaultAppPool`

1. Fate clic sul pulsante Nomi di controllo. Quando Windows risolve l&#39;account utente, fate clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella dispatcher, selezionate l&#39;account appena aggiunto, abilitate tutte le autorizzazioni per l&#39;account** tranne Controllo completo**, quindi fate clic su OK. Fate clic su OK per chiudere la finestra di dialogo Proprietà cartella.
1. Use a text editor to open the `disp_iis.ini` file.
1. Aggiungete una riga di testo simile all&#39;esempio seguente per configurare la posizione del file di registro e salvare il file:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Next Steps {#next-steps}

Prima di iniziare a usare il dispatcher è necessario:

* [Configura](dispatcher-configuration.md) dispatcher
* [Corsivo AEM](page-invalidate.md) per lavorare con Dispatcher.

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>Instructions for installation under both **Windows** and **Unix** are covered here. Prestate attenzione durante la procedura.

### Installing Apache Web Server {#installing-apache-web-server}

For Information about how to install an Apache Web Server read the installation manual - either [online](https://httpd.apache.org/) or in the distribution.

>[!CAUTION]
>
>If you are creating an Apache binary by compiling the source files, make sure that you turn on **dynamic modules support**. This can be done by using any of the **--enable-shared** options. At a minimum include the `mod_so` module.
>
>Ulteriori informazioni sono disponibili nel manuale di installazione di Apache Web Server.

Also see the Apache HTTP Server [Security Tips](https://httpd.apache.org/docs/2.4/misc/security_tips.html) and [Security Reports](https://httpd.apache.org/security_report.html).

### Apache Web Server - Add the Dispatcher Module {#apache-web-server-add-the-dispatcher-module}

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
      Copy `dispatcher-apache<options>.so` into this directory.\
      To simplify long-term maintenance you can also create a symbolic link named `mod_dispatcher.so` to the Dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Copy the dispatcher.any file to the `<APACHE_ROOT>/conf` directory.

   **Nota:** Potete posizionare questo file in un percorso diverso, purché la proprietà dispatcherlog del modulo Dispatcher sia configurata di conseguenza. (Vedi Voci di configurazione specifiche per dispatcher sotto).

### Apache Web Server - Configure SELinux Properties {#apache-web-server-configure-selinux-properties}

Se si esegue Dispatcher su redhat Linux Kernel 2.6 con selinux abilitato, è possibile che venga visualizzato in messaggi di errore come nel logfile di registro del dispatcher.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Ciò è probabilmente dovuto a una protezione selinux abilitata. Quindi, è necessario effettuare le seguenti operazioni:

* Configurate il contesto selinux del file del modulo di dispatcher.
* Abilitare gli script e i moduli HTTPD per effettuare connessioni di rete.
* Configurate il contesto selinux del docroot, dove vengono memorizzati i file memorizzati nella cache.

Enter the following commands in a terminal window, replacing `[path to the dispatcher.so file]` with the path to the Dispatcher module that you installed to Apache Web Server, and *`path to the docroot`* with the path where the docroot is located (e.g. `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - Configure Apache Web Server for Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

The Apache Web Server needs to be configured, using `httpd.conf`. In the Dispatcher installation kit you will find an example configuration file named `httpd.conf.disp<x>`.

Questi passaggi sono obbligatori:

1. Accedi a `<APACHE_ROOT>/conf`.
1. Open `httpd.conf`for editing.
1. Nell&#39;elenco elencato è necessario aggiungere le seguenti voci di configurazione:

   * **Loadmodule** per caricare il modulo all&#39;avvio.
   * Dispatcher-specific configuration entries, including **DispatcherConfig, DispatcherLog** and **DispatcherLogLevel**.
   * **Sethandler** per attivare il dispatcher. **Loadmodule**.
   * **Modmimeusepathinfo** per configurare il comportamento **di mod_ mime**.

1. (Facoltativo) Si consiglia di modificare il proprietario della directory htdocs:

   * Il server apache inizia come radice, ma i processi secondari vengono avviati come daemon (a scopo di sicurezza). The DocumentRoot (`<APACHE_ROOT>/htdocs`) must belong to the user daemon:

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**Loadmodule**

Nella tabella seguente sono riportati alcuni esempi d&#39;uso; le voci esatte sono in base al server Web Apache specifico:

|  |  |
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
| Dispatcherloglevel | Log level for the log file: <br/>0 - Errors <br/>1 - Warnings <br/>2 - Infos <br/>3 - Debug <br/>**Note**: It is recommended to set the log level to 3 during installation and testing, then to 0 when running in a production environment. |
| Dispatchernoserverheader | *Questo parametro è obsoleto e non ha più effetto.*<br/><br/> Definisce l&#39;Intestazione server da usare: <br/><ul><li>undefined o 0 - L&#39;intestazione del server HTTP contiene la versione AEM. </li><li>1 - L&#39;intestazione server Apache viene utilizzata.</li></ul> |
| Dispatcherweakeroot | Defines whether to decline requests to the root &quot;/&quot;: <br/>**0** - accept requests to / <br/>**1** - requests to / are not handled by the dispatcher; use mod_alias for the correct mapping. |
| Dispatcheruseprocessedurl | Defines whether to use pre-processed URLs for all further processing by Dispatcher: <br/>**0** - use the original URL passed to the web server. <br/>**1** - Il dispatcher utilizza l&#39;URL già elaborato dai gestori che precedono il dispatcher ( `mod_rewrite`ovvero) anziché l&#39;URL originale passato al server Web. Ad esempio, l&#39;originale o l&#39;URL elaborato viene associato ai filtri del dispatcher. L&#39;URL viene utilizzato anche come base per la struttura del file della cache. Per informazioni su mod_ rewrite, consultate la documentazione del sito Apache ad esempio Apache 2.4. Quando si utilizza mod_ rewrite, è consigliabile utilizzare il flag «passthrough» | PT &#39;(passa al gestore successivo) per forzare il fatto che il motore di riscrittura imposti il campo uri della struttura request_ rec interna sul valore del campo nome file. |
| Dispatcherpasserror | Defines how to support error codes for ErrorDocument handling: <br/>**0** - Dispatcher spools all error responses to the client. <br/>**1** - Il dispatcher non esegue lo spool di una risposta di errore al client (dove il codice di stato è maggiore o uguale a 400), ma trasmette il codice di stato ad Apache, ad es. consente a una direttiva errordocument di elaborare tale codice di stato. <br/>**Intervallo** di codice - Specifica una serie di codici di errore per i quali la risposta viene passata ad Apache. Altri codici di errore vengono passati al client. Ad esempio, la seguente configurazione trasmette le risposte per l&#39;errore 412 al client, e tutti gli altri errori vengono passati ad Apache: Dispatcherpasserror 400-411,413-417 |
| Dispatcherkeepalivetimeout | Specifica il timeout &quot;keep-alive&quot;, in secondi. A partire da Dispatcher 4.2.0 il valore predefinito di mantenimento è 60. Un valore pari a 0 disattiva il ciclo di vita. |
| Dispatchernocanonurl | Impostando questo parametro su Attivato, l&#39;URL non elaborato viene trasferito al backend anziché quello canonico e le impostazioni di dispatcheruseprocessedurl ignoreranno. Il valore predefinito è Disattivato. <br/>**Nota**: Le regole di filtro nella configurazione del dispatcher saranno sempre valutate rispetto all&#39;URL sanitized, non all&#39;URL non elaborato. |




>[!NOTE]
>
>Le voci percorso sono relative alla directory principale del server Web Apache.

>[!NOTE]
>
>The default settings for the Server Header are: `  
ServerTokens Full` `  
DispatcherNoServerHeader 0`\
che mostra la versione di AEM (a scopi statistici). If you want to disable such information being available in the header you can set: `  
ServerTokens Prod`\
See the [Apache Documentation about ServerTokens Directive (for example, for Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html) for more information.

**Sethandler**

After these entries you must add a **SetHandler** statement to the context of your configuration ( `<Directory>`, `<Location>`) for the Dispatcher to handle the incoming requests. L&#39;esempio seguente configura il dispatcher per gestire le richieste del sito Web completo:

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
The parameter of the **SetHandler** statement must be written *exactly as in the above examples*, as this is the name of the handler defined in the module.
Per informazioni dettagliate su questo comando, consultate i file di configurazione forniti e la documentazione Apache Web Server.

**Modmimeusepathinfo**

After the **SetHandler** statement you should also add the **ModMimeUsePathInfo** definition.

>[!NOTE]
The `ModMimeUsePathInfo` parameter should only be used and configured if you are using Dispatcher version 4.0.9, or higher.
(Nota: la versione 4.0.9 del dispatcher è stata rilasciata nel 2011. Se utilizzi una versione precedente, è opportuno effettuare l&#39;aggiornamento a una versione di Dispatcher recente.

The **ModMimeUsePathInfo** parameter should be set `On` for all Apache configurations:

`ModMimeUsePathInfo On`

The mod_mime module (see for example, [Apache Module mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) is used to assign content metadata to the content selected for an HTTP response. La configurazione predefinita significa che quando mod_ mime determina il tipo di contenuto, viene considerata solo la porzione dell&#39;URL che viene mappata su un file o una directory.

When `On`, the `ModMimeUsePathInfo` parameter specifies that `mod_mime` is to determine the content type based on the *complete* URL; this means that virtual resources will have metainformation applied based on their extension.

The following example activates **ModMimeUsePathInfo**:

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

### Enable Support for HTTPS (Unix and Linux) {#enable-support-for-https-unix-and-linux}

Il dispatcher utilizza openssl per implementare comunicazioni sicure attraverso HTTP. Starting from Dispatcher version **4.2.0**, OpenSSL 1.0.0 and OpenSSL 1.0.1 are supported. Per impostazione predefinita, il dispatcher utilizza openssl 1.0.0. Per utilizzare openssl 1.0.1, utilizzate la seguente procedura per creare collegamenti simbolici in modo che il dispatcher utilizzi le librerie openssl installate.

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
If you are using a customized version of Apache, make sure Apache and Dispatcher are compiled using the same version of [OpenSSL](https://www.openssl.org/source/).

### Next Steps {#next-steps-1}

Prima di iniziare a utilizzare il dispatcher è necessario:

* [Configura](dispatcher-configuration.md) dispatcher
* [Corsivo AEM](page-invalidate.md) per lavorare con Dispatcher.

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
Le istruzioni per gli ambienti Windows e Unix sono descritte qui.
Prestate attenzione quando selezionate quali eseguire.

### Sun Java System Web Server / iPlanet - Installing your Web Server {#sun-java-system-web-server-iplanet-installing-your-web-server}

Per informazioni dettagliate su come installare tali server Web, consultate la relativa documentazione:

* Sun Java System Web Server
* Iplanet Web Server

### Sun Java System Web Server / iPlanet - Add the Dispatcher Module {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

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

1. Place the Dispatcher file in the web server&#39;s `plugin` directory:

### Sun Java System Web Server / iPlanet - Configure for the Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

The web server needs to be configured, using `obj.conf`. In the Dispatcher installation kit you will find an example configuration file named `obj.conf.disp`.

1. Accedi a `<WEBSERVER_ROOT>/config`.
1. Open `obj.conf`for editing.
1. Copia la linea che inizia:\
   `Service fn="dispService"`\
   from `obj.conf.disp` to the initialization section of `obj.conf`.

1. Salvate le modifiche.
1. Open `magnus.conf` for editing.
1. Copiare le due righe che iniziano:\
   `Init funcs="dispService, dispInit"`\
   e\
   `Init fn="dispInit"`\
   from `obj.conf.disp` to the initialization section of `magnus.conf`.

1. Salvate le modifiche.

>[!NOTE]
The following configurations should all be on one line and the `$(SERVER_ROOT)` and `$(PRODUCT_SUBDIR)` must be replaced by the respective values.

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
| config | Location and name of the configuration file `dispatcher.any.` |
| log, logfile | Posizione e nome del file di registro. |
| loglevel | Log level for when writing messages to the log file: <br/>**0** Errors <br/>**1** Warnings <br/>**2** Infos <br/>**3** Debug <br/>**Note:** It is recommended to set the log level to 3 during installation and testing and to 0 when running in a production environment. |
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

### Next Steps {#next-steps-2}

Prima di iniziare a utilizzare il dispatcher è necessario:

* [Configura](dispatcher-configuration.md) dispatcher
* [Corsivo AEM](page-invalidate.md) per lavorare con Dispatcher.
