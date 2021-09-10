---
title: Installazione di Dispatcher
seo-title: Installazione di AEM Dispatcher
description: Scopri come installare il modulo Dispatcher su Microsoft Internet Information Server, server web Apache e Sun Java Web Server-iPlanet.
seo-description: Scopri come installare il modulo AEM Dispatcher su Microsoft Internet Information Server, server web Apache e Sun Java Web Server-iPlanet.
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 35739785aa835a0b995fab4710a0e37bd0ff62b4
workflow-type: ht
source-wordcount: '3684'
ht-degree: 100%

---

# Installazione di Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

Utilizza la pagina [Note sulla versione di Dispatcher](release-notes.md) per ottenere il file di installazione di Dispatcher più recente per il tuo sistema operativo e server web. I numeri di versione di Dispatcher sono indipendenti quelli di Adobe Experience Manager e sono compatibili con le versioni di Adobe Experience Manager 6.x, 5.x e Adobe CQ 5.x.

>[!NOTE]
>
>Considera che Adobe Experience Manager 6.5 richiede Dispatcher versione 4.3.2 o successiva. Ciò detto, le versioni di Dispatcher sono indipendenti da AEM, ad esempio, Dispatcher versione 4.3.2 è compatibile anche con Adobe Experience Manager 6.4.

Viene utilizzata la seguente convenzione per la denominazione dei file:

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

Ad esempio, il file `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` contiene Dispatcher versione 4.3.1 per un server web Apache 2.4 che viene eseguito su Linux i686 e compattato utilizzando il formato **tar**.

La tabella che segue riporta gli identificativi utilizzati nei nomi dei file per ciascun server web:

| Server web | Kit di installazione |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;altri parametri> |
| Microsoft Internet Information Server 7.5, 8, 8.5 | dispatcher-**iis**-&lt;altri parametri> |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;altri parametri> |

>[!CAUTION]
>
>Installa la versione più recente di Dispatcher disponibile per la tua piattaforma. È opportuno aggiornare l’istanza di Dispatcher ogni anno per utilizzare sempre la versione più recente e sfruttarne i miglioramenti apportati.

Ogni archivio contiene i seguenti file:

* Moduli di Dispatcher
* Un esempio di file di configurazione
* Il file README contenente le istruzioni di installazione e le informazioni dell’ultimo minuto
* Il file CHANGES in cui sono elencati i problemi risolti in questa e nelle precedenti versioni

>[!NOTE]
>
>Prima di iniziare l’installazione, leggi il file README per eventuali modifiche dell’ultimo minuto o note specifiche per la tua piattaforma.

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

Per informazioni su come installare questo server web, vedi le risorse seguenti:

* Documentazione di Microsoft Internet Information Server
* [&quot;Sito ufficiale Microsoft IIS&quot;](https://www.iis.net/)

### Componenti IIS richiesti {#required-iis-components}

Le versioni 8.5 e 10 di IIS richiedono l’installazione dei seguenti componenti IIS:

* Estensioni ISAPI

Inoltre, è necessario aggiungere il ruolo Server web (IIS). Utilizza Server Manager per aggiungere ruolo e componenti.

## Microsoft IIS: installazione del modulo Dispatcher {#microsoft-iis-installing-the-dispatcher-module}

L’archivio richiesto per Microsoft Internet Information System è:

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

Il file ZIP contiene i seguenti file:

| File | Descrizione |
|--- |--- |
| `disp_iis.dll` | File DLL (Dynamic Link Library) di Dispatcher. |
| `disp_iis.ini` | File di configurazione per IIS. Questo esempio può essere aggiornato in base alle tue esigenze. **Nota**: il file ini deve avere lo stesso nome-radice del file DLL. |
| `dispatcher.any` | Esempio di file di configurazione per Dispatcher. |
| `author_dispatcher.any` | Un esempio di file di configurazione per Dispatcher che funziona con l’istanza Autore. |
| README | File Readme contenente istruzioni di installazione e informazioni dell’ultimo minuto. **Nota**: leggere questo file prima di iniziare l’installazione. |
| CHANGES | Il file Changes elenca i problemi risolti in questa e nelle precedenti versioni. |

Per copiare i file Dispatcher nella posizione corretta, utilizza la procedura descritta di seguito.

1. Utilizza Esplora risorse per creare la directory `<IIS_INSTALLDIR>/Scripts`, ad esempio `C:\inetpub\Scripts`.

1. Estrai i seguenti file dal pacchetto di Dispatcher in questa directory Scripts:

   * `disp_iis.dll`
   * `disp_iis.ini`
   * Uno dei file seguenti, a seconda che Dispatcher funzioni con un’istanza Autore AEM o AEM Publish:
      * Istanza Autore: `author_dispatcher.any`
      * Istanza Publish: `dispatcher.any`

## Microsoft IIS: configurare il file INI di Dispatcher {#microsoft-iis-configure-the-dispatcher-ini-file}

Modifica il file `disp_iis.ini` per configurare l’installazione di Dispatcher. Il formato di base del file `.ini` è il seguente:

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

La tabella che segue descrive le singole proprietà.

| Parametro | Descrizione |
|--- |--- |
| configpath | La posizione di `dispatcher.any` all’interno del file system locale (percorso assoluto). |
| logfile | Posizione del file `dispatcher.log`. Se questo parametro non è impostato, i messaggi del registro vanno nel registro eventi di Windows. |
| loglevel | Definisce il livello di registro utilizzato per inviare messaggi al registro eventi. È possibile specificare i seguenti valori: livello del file di registro: <br/>0 - solo messaggi di errore. <br/>1 - errori e avvisi. <br/>2 - errori, avvisi e messaggi informativi <br/>3 - errori, avvisi, messaggi informativi e di debug. <br/>**Nota**: si consiglia di impostare il livello di registro su 3 durante l’installazione e il test, quindi su 0 quando l’esecuzione avviene in un ambiente di produzione. |
| replaceauthorization | Specifica come vengono gestite le intestazioni di autorizzazione nella richiesta HTTP. Sono validi i seguenti valori:<br/>0 - Le intestazioni di autorizzazione non vengono modificate. <br/>1 - Sostituisce qualsiasi intestazione denominata &quot;Authorization&quot; diversa da &quot;Basic&quot; con il suo `Basic <IIS:LOGON\_USER>` equivalente.<br/> |
| servervariables | Definisce il modo in cui vengono elaborate le variabili del server.<br/>0 - Le variabili del server IIS non vengono inviate né a Dispatcher né ad AEM. <br/>1 - Tutte le variabili del server IIS (come `LOGON\_USER, QUERY\_STRING, ...`) vengono inviate a Dispatcher, insieme alle intestazioni delle richieste (e anche all’istanza di AEM, se non memorizzate in cache).  <br/>Le variabili del server includono `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` e molte altre. Vedi la documentazione di IIS per l’elenco completo delle variabili con relativi dettagli. |
| enable_chunked_transfer | Definisce se abilitare (1) o disabilitare (0) il trasferimento a blocchi per la risposta del client. Il valore predefinito è 0. |

Esempio di configurazione:

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Configurazione di Microsoft IIS {#configuring-microsoft-iis}

Configura IIS in modo da integrare il modulo ISAPI di Dispatcher. In IIS si utilizza il mapping di applicazioni con caratteri jolly.

### Configurazione dell’accesso anonimo: IIS 8.5 e 10 {#configuring-anonymous-access-iis-and}

L’agente di replica Flush predefinito sull’istanza Autore è configurato in modo da non inviare credenziali di sicurezza con le richieste di flushing. Pertanto, il sito web che utilizzi come cache di Dispatcher deve consentire l’accesso anonimo.

Se il sito web utilizza un metodo di autenticazione, l’agente di replica Flush deve essere configurato di conseguenza.

1. Apri IIS Manager e seleziona il sito web che utilizzi come cache di Dispatcher.
1. Utilizzando la modalità Visualizzazione funzionalità, nella sezione IIS fai doppio clic su Autenticazione.
1. Se l’autenticazione anonima non è abilitata, seleziona Autenticazione anonima e nell’area Azioni fai clic su Abilita.

### Integrazione del modulo ISAPI di Dispatcher: IIS 8.5 e 10 {#integrating-the-dispatcher-isapi-module-iis-and}

Per aggiungere il modulo ISAPI di Dispatcher a IIS, utilizza la procedura descritta di seguito.

1. Apri IIS Manager.
1. Seleziona il sito web che utilizzi come cache di Dispatcher.
1. Utilizzando la modalità Visualizzazione funzionalità, nella sezione IIS fai doppio clic su Mapping gestori.
1. Nel pannello Azioni della pagina Mapping gestori, fai clic su Aggiungi mapping di script con caratteri jolly, inserisci i seguenti valori delle proprietà e fai clic su OK:

   * Percorso richiesta: *
   * Eseguibile: percorso assoluto del file disp_iis.dll, ad esempio `C:\inetpub\Scripts\disp_iis.dll`.
   * Nome: un nome descrittivo per il mapping del gestore, ad esempio `Dispatcher`.

1. Nella finestra di dialogo visualizzata, per aggiungere la libreria disp_iis.dll all’elenco delle restrizioni ISAPI e CGI, fai clic su Sì.

   Per IIS 7.0 e 7.5 la configurazione è completa. Se stai configurando IIS 8.0, procedi con i passaggi successivi.

1. (IIS 8.0) Nell’elenco dei mapping dei gestori, seleziona quello appena creato e nell’area Azioni fai clic su Modifica.
1. (IIS 8.0) Nella finestra di dialogo Modifica mapping di script, fai clic sul pulsante Restrizioni richieste.
1. (IIS 8.0) Per garantire che il gestore sia utilizzato per i file e le cartelle che non sono ancora memorizzati in cache, deseleziona Richiama gestore solo se la richiesta è mappata a, quindi fai clic su OK.
1. (IIS 8.0) Nella finestra di dialogo Modifica mapping di script, fai clic su OK.

### Configurazione dell’accesso alla cache: IIS 8.5 e 10 {#configuring-access-to-the-cache-iis-and}

Fornisci all’utente del pool di applicazioni predefinito l’accesso in scrittura alla cartella che viene utilizzata come cache di Dispatcher.

1. Fai clic con il pulsante destro del mouse sulla cartella principale del sito web che utilizzi come cache di Dispatcher e fai clic su Proprietà, ad esempio `C:\inetpub\wwwroot`.
1. Fai clic su Modifica nella scheda Sicurezza, quindi fai clic su Aggiungi nella finestra di dialogo Autorizzazioni. Si apre una finestra di dialogo per la selezione degli account utente. Fai clic sul pulsante Percorsi, seleziona il nome del computer e fai clic su OK.

   Tieni aperta questa finestra di dialogo mentre completi il passaggio successivo.

1. In IIS Manager, seleziona il sito IIS che utilizzi come cache di Dispatcher e fai clic su Impostazioni avanzate sul lato destro della finestra.
1. Seleziona il valore della proprietà Application Pool e copiala negli Appunti.
1. Torna alla finestra di dialogo già aperta. Nella casella Immettere i nomi degli oggetti da selezionare, digita `IIS AppPool\`, quindi incolla il contenuto degli Appunti. Il valore deve essere simile al seguente:

   `IIS AppPool\DefaultAppPool`

1. Fai clic sul pulsante Controlla nomi. Quando Windows risolve l’account utente, fai clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella di Dispatcher, seleziona l’account appena aggiunto, abilita tutte le autorizzazioni per l’account **eccetto Controllo completo** e fai clic su OK. Fai clic su OK per chiudere la finestra di dialogo Proprietà.

### Registrazione del tipo MIME JSON: IIS 8.5 e 10 {#registering-the-json-mime-type-iis-and}

Utilizza la seguente procedura per registrare il tipo MIME JSON quando vuoi che Dispatcher consenta le chiamate JSON.

1. In IIS Manager, seleziona il sito Web e, utilizzando Visualizzazione funzionalità, fai doppio clic su Tipi MIME.
1. Se l’estensione JSON non è presente nell’elenco, nel pannello Azioni fai clic su Aggiungi, immetti i seguenti valori delle proprietà, quindi fai clic su OK:

   * Estensione nome file: `.json`
   * Tipo MIME: `application/json`

### Rimozione del segmento nascosto bin: IIS 8.5 e 10 {#removing-the-bin-hidden-segment-iis-and}

Per rimuovere il segmento nascosto `bin`, utilizza la procedura descritta di seguito. I siti Web che non sono nuovi possono contenere questo segmento nascosto.

1. In IIS Manager, seleziona il sito Web e, utilizzando Visualizzazione funzionalità, fai doppio clic su Filtro richieste.
1. Seleziona il segmento `bin`, fai clic su Rimuovi e nella finestra di dialogo di conferma fai clic su Sì.

### Registrazione dei messaggi IIS in un file: IIS 8.5 e 10 {#logging-iis-messages-to-a-file-iis-and}

Utilizza la procedura seguente per scrivere i messaggi di registro di Dispatcher in un file di registro invece che nel registro eventi di Windows. È necessario configurare Dispatcher in modo che utilizzi il file di registro e fornire a IIS l’accesso in scrittura al file.

1. Utilizza Esplora risorse per creare una cartella denominata `dispatcher` sotto la cartella dei registri dell’installazione di IIS. Il percorso di questa cartella per un’installazione tipica è `C:\inetpub\logs\dispatcher`.

1. Fai clic con il pulsante destro del mouse sulla cartella di Dispatcher e fai clic su Proprietà.
1. Fai clic su Modifica nella scheda Sicurezza, quindi fai clic su Aggiungi nella finestra di dialogo Autorizzazioni. Si apre una finestra di dialogo per la selezione degli account utente. Fai clic sul pulsante Percorsi, seleziona il nome del computer e fai clic su OK.

   Tieni aperta questa finestra di dialogo mentre completi il passaggio successivo.

1. In Gestione IIS, seleziona il sito IIS che utilizzi come cache di Dispatcher e fai clic su Impostazioni avanzate sul lato destro della finestra.
1. Seleziona il valore della proprietà Application Pool e copiala negli Appunti.
1. Torna alla finestra di dialogo già aperta. Nella casella Immettere i nomi degli oggetti da selezionare, digita `IIS AppPool\`, quindi incolla il contenuto degli Appunti. Il valore deve essere simile al seguente:

   `IIS AppPool\DefaultAppPool`

1. Fai clic sul pulsante Controlla nomi. Quando Windows risolve l’account utente, fai clic su OK.
1. Nella finestra di dialogo Autorizzazioni per la cartella di Dispatcher, seleziona l’account appena aggiunto, abilita tutte le autorizzazioni per l’account **eccetto Controllo completo** e fai clic su OK. Fai clic su OK per chiudere la finestra di dialogo Proprietà.
1. Utilizza un editor di testo per aprire il file `disp_iis.ini`.
1. Aggiungi una riga di testo simile all’esempio seguente per configurare il percorso del file di registro, quindi salva il file:

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Passaggi successivi {#next-steps}

Prima di poter iniziare a utilizzare Dispatcher è necessario conoscere:

* [Configurare](dispatcher-configuration.md) Dispatcher
* [Configurare AEM](page-invalidate.md) per funzionare con Dispatcher.

## Server web Apache {#apache-web-server}

>[!CAUTION]
>
>Le istruzioni per l’installazione in **Windows** e **Unix** sono riportate qui. Fai molta durante l’esecuzione di questi passaggi.

### Installazione del server web Apache {#installing-apache-web-server}

Per informazioni sull’installazione di un server web Apache, leggere il manuale di installazione [online](https://httpd.apache.org/) o nella distribuzione.

>[!CAUTION]
>
>Se stai creando un file binario di Apache tramite la compilazione dei file source, accertati che sia ttivo il **supporto dei moduli dinamici**. Per questo scopo, puoi utilizzare una qualsiasi delle opzioni **--enable-shared**. Quantomeno includi il modulo `mod_so`.
>
>Ulteriori informazioni sono disponibili nel manuale di installazione del server web Apache.

Vedi anche i [suggerimenti sulla sicurezza](https://httpd.apache.org/docs/2.4/misc/security_tips.html) e i [rapporti sulla sicurezza](https://httpd.apache.org/security_report.html) di Apache HTTP Server.

### Server web Apache: aggiungi il modulo Dispatcher {#apache-web-server-add-the-dispatcher-module}

Dispatcher viene fornito per:

* **Windows**: una DLL (Dynamic Link Library)
* oppure **Unix**: un DSO (Dynamic Shared Object)

L’archivio di installazione contiene i seguenti file, a seconda che sia stato selezionato Windows o Unix:

| File | Descrizione |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows: il file DLL di Dispatcher. |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | Unix: il file DSO di Dispatcher. |
| mod_dispatcher.so | Unix: un esempio di collegamento. |
| http.conf.disp&lt;x> | Un esempio di file di configurazione del server Apache. |
| dispatcher.any | Esempio di file di configurazione per Dispatcher. |
| README | File Readme contenente istruzioni di installazione e informazioni dell’ultimo minuto. **Nota**: leggere questo file prima di iniziare l’installazione. |
| CHANGES | Il file Changes in cui sono elencati i problemi risolti in questa e nelle precedenti versioni. |

Per aggiungere Dispatcher al server web Apache, fai quanto segue:

1. Posiziona il file di Dispatcher nella directory appropriata del modulo Apache:

   * **Windows**: posiziona `disp_apache<x.y>.dll` in `<APACHE_ROOT>/modules`
   * **Unix**: individua la directory `<APACHE_ROOT>/libexec` o `<APACHE_ROOT>/modules`in base al tipo di installazione.\
      Copia il file `dispatcher-apache<options>.so` in questa directory.\
      Per semplificare la manutenzione a lungo termine, puoi anche creare un collegamento simbolico denominato `mod_dispatcher.so` a Dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Copia il file dispatcher.any nella directory `<APACHE_ROOT>/conf`.

   **Nota:** puoi posizionare il file in una posizione diversa, purché la proprietà DispatcherLog del modulo Dispatcher sia configurata di conseguenza. (Vedi qui di seguito Voci di configurazione specifiche per Dispatcher).

### Server web Apache: configura le proprietà SELinux {#apache-web-server-configure-selinux-properties}

Se esegui Dispatcher su RedHat Linux Kernel 2.6 con SELinux abilitato, potresti avere messaggi di errore come questo nel file di registro di Dispatcher.

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

Ciò è probabilmente dovuto a una qualche opzione di sicurezza abilitata in SELinux. Quindi devi effettuare le seguenti attività:

* Configura il contesto SELinux del file del modulo di Dispatcher.
* Abilita gli script e i moduli HTTPD per effettuare le connessioni di rete.
* Configura il contesto SELinux della directory principale dei documenti, nella quale vengono archiviati i file memorizzati in cache.

Immettere i seguenti comandi in una finestra del terminale, sostituendo `[path to the dispatcher.so file]` con il percorso del modulo di Dispatcher installato sul server web Apache e *`path to the docroot`* con il percorso in cui si trova la directory principale dei documenti (ad esempio, `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Server web Apache: configurazione del server web Apache per Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

È necessario configurare il server web Apache utilizzando il file `httpd.conf`. Il kit di installazione di Dispatcher contiene un esempio di file di configurazione denominato `httpd.conf.disp<x>`.

I seguenti passaggi sono obbligatori:

1. Accedi a `<APACHE_ROOT>/conf`.
1. Apri `httpd.conf` per la modifica.
1. È necessario aggiungere le seguenti voci di configurazione, nell’ordine indicato:

   * **LoadModule** per caricare il modulo all’avvio.
   * Voci di configurazione specifiche per Dispatcher, tra cui **DispatcherConfig, DispatcherLog** e **DispatcherLogLevel**.
   * **SetHandler** per attivare Dispatcher. **LoadModule**.
   * **ModMimeUsePathInfo** per configurare il comportamento di **mod_mime**.

1. (Facoltativo) È consigliabile modificare il proprietario della directory htdocs:

   * Il server Apache viene avviato come radice, anche se i processi figlio iniziano come daemon (per motivi di sicurezza). DocumentRoot (`<APACHE_ROOT>/htdocs`) deve appartenere al daemon utente:

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

La tabella che segue riporta alcuni esempi che possono essere utilizzati; le voci esatte dipendono dal tipo di server web Apache in uso:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix (presupponendo un collegamento simbolico) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>Il primo parametro di ogni istruzione deve essere scritto esattamente come negli esempi precedenti.
>
>Per informazioni dettagliate su questo comando, vedi gli esempi di file di configurazione forniti e la documentazione del server web Apache.

**Voci di configurazione specifiche per Dispatcher**

Le voci di configurazione specifiche per Dispatcher vanno inserite dopo la voce LoadModule. La tabella che segue riporta un esempio di configurazione applicabile sia per Unix che per Windows:

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
| DispatcherConfig | Posizione e nome del file di configurazione di Dispatcher. <br/>Quando questa proprietà si trova nella configurazione del server principale, tutti gli host virtuali ereditano il valore della proprietà. Tuttavia, gli host virtuali possono includere la proprietà DispatcherConfig per ignorare la configurazione del server principale. |
| DispatcherLog | Posizione e nome del file di registro. |
| DispatcherLogLevel | Livello del file di registro: <br/>0 - Errori <br/>1 - Avvisi <br/>2 - Informazioni <br/>3 - Debug <br/>**Nota**: si consiglia di impostare il livello del file di registro su 3 durante l’installazione e il test, quindi su 0 quando l’esecuzione avviene in un ambiente di produzione. |
| DispatcherNoServerHeader | *Questo parametro è obsoleto e non è più valido.*<br/><br/> Definisce l’intestazione del server da utilizzare: <br/><ul><li>indefinito oppure 0 - l’intestazione del server HTTP contiene la versione di AEM. </li><li>1 - Viene utilizzata l’intestazione del server Apache.</li></ul> |
| DispatcherDeclineRoot | Definisce se rifiutare le richieste alla radice &quot;/&quot;: <br/>**0** - Accetta le richieste a / <br/>**1** - Le richieste a / non sono gestite da Dispatcher; utilizza mod_alias per il mapping corretto. |
| DispatcherUseProcessedURL | Definisce se utilizzare URL preelaborati per tutte le ulteriori elaborazioni da parte di Dispatcher: <br/>**0** - Utilizza l’URL originale inviato al server web. <br/>**1** - Dispatcher utilizza l’URL già elaborato dagli handler che precedono Dispatcher (vale a dire, `mod_rewrite`) invece dell’URL originale inviato al server web.  Ad esempio, l’URL originale o quello elaborato corrisponde ai filtri di Dispatcher. L’URL viene utilizzato anche come base per la struttura del file della cache.   Per informazioni su mod_rewrite, vedi la documentazione del sito web di Apache; ad esempio, Apache 2.4. Quando si utilizza mod_rewrite, è consigliabile utilizzare il flag ‘passthrough | PT’ (passare all’handler successivo) per forzare il motore di riscrittura a impostare il campo URI della struttura request_rec interna sul valore del campo filename. |
| DispatcherPassError | Definisce come supportare i codici di errore per la gestione di ErrorDocument: <br/>**0** - Dispatcher effettua lo spool di tutte le risposte di errore inviate al client. <br/>**1** - Dispatcher non effettua lo spool di una risposta di errore inviata al client (nel caso in cui il codice di stato sia maggiore di o uguale a 400), ma invia il codice di stato ad Apache, il quale, ad esempio, consente a una direttiva ErrorDocument di elaborarlo. <br/>**Intervallo di codici** - Specifica un intervallo di codici di errore per i quali la risposta viene inviata ad Apache. Gli altri codici di errore vengono inviati al client. Ad esempio, la seguente configurazione invia al client le risposte per l’errore 412 e tutti gli altri errori vengono inviati ad Apache: DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | Specifica il timeout keep-alive espresso in secondi. A partire dalla versione 4.2.0 di Dispatcher, il valore keep-alive predefinito è 60. Il valore 0 disattiva keep-alive. |
| DispatcherNoCanonURL | Se si attiva questo parametro, l’URL non elaborato viene inviato al backend invece che alla destinazione canonica e vengono ignorate tutte le impostazioni di DispatcherUseProcessedURL. Il valore predefinito è Disattivato. <br/>**Nota**: le regole dei filtri nella configurazione di Dispatcher verranno sempre valutate in base all’URL bonificato e non all’URL non elaborato. |




>[!NOTE]
>
>Le voci del percorso sono relative alla directory principale del server web Apache.

>[!NOTE]
>
>Le impostazioni predefinite per l’intestazione del server sono:
>
>`ServerTokens Full`
>
>`DispatcherNoServerHeader 0`
>
>Mostra la versione di AEM (per fini statistici). Se vuoi disattivare la disponibilità di queste informazioni nell’intestazione, puoi impostare:
>
>`ServerTokens Prod`
>
>Per ulteriori informazioni, vedi la [Documentazione di Apache sulla direttiva ServerTokens (ad esempio, per Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html).

**SetHandler**

Dopo queste voci è necessario aggiungere un’istruzione **SetHandler** al contesto della configurazione (`<Directory>`, `<Location>`) affinché Dispatcher possa gestire le richieste in ingresso. L’esempio che segue configura Dispatcher per gestire le richieste dell’intero sito web:

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

L’esempio che segue configura Dispatcher per gestire le richieste di un dominio virtuale:

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
>Il parametro dell’istruzione **SetHandler** deve essere scritto *esattamente come negli esempi precedenti*, in quanto questo è il nome dell’handler definito nel modulo.
>
>Per informazioni dettagliate su questo comando, vedi gli esempi di file di configurazione forniti e la documentazione del server web Apache.

**ModMimeUsePathInfo**

Dopo l’istruzione **SetHandler** è necessario aggiungere anche la definizione **ModMimeUsePathInfo**.

>[!NOTE]
>
>Il parametro `ModMimeUsePathInfo` va utilizzato e configurato solo se utilizzi Dispatcher versione 4.0.9 o successive.
>
>(Dispatcher versione 4.0.9 è stato rilasciato nel 2011. Se utilizzi una versione precedente, sarebbe auspicabile l’aggiornamento a una versione recente di Dispatcher).

Il parametro **ModMimeUsePathInfo** va impostato su `On` per tutte le configurazioni di Apache:

`ModMimeUsePathInfo On`

Il modulo mod_mime (vedi, ad esempio, [Modulo Apache mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) viene utilizzato per assegnare metadati di contenuto al contenuto selezionato per una risposta HTTP. La configurazione predefinita comporta che quando mod_mime determina il tipo di contenuto, verrà considerata solo la parte dell’URL mappata su un file o una directory.

Se attivato (`On`), il parametro `ModMimeUsePathInfo` specifica che `mod_mime` deve determinare il tipo di contenuto in base all’URL *completo*; ciò significa che le risorse virtuali avranno metadati applicati in base alla loro estensione.

L’esempio che segue attiva **ModMimeUsePathInfo**:

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

### Abilitazione del supporto per HTTPS (Unix e Linux) {#enable-support-for-https-unix-and-linux}

Dispatcher utilizza OpenSSL per implementare una comunicazione sicura tramite HTTP. A partire dalla versione di Dispatcher **4.2.0**, sono supportati OpenSSL 1.0.0 e OpenSSL 1.0.1. Dispatcher utilizza OpenSSL 1.0.0 per impostazione predefinita. Per utilizzare OpenSSL 1.0.1, segui la procedura sotto riportata per creare collegamenti simbolici in modo che Dispatcher possa utilizzare le librerie OpenSSL installate.

1. Apri un terminale e cambia la directory corrente con la directory in cui sono installate le librerie OpenSSL, ad esempio:

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
>Se utilizzi una versione personalizzata di Apache, accertati che Apache e Dispatcher siano compilati con la stessa versione di [OpenSSL](https://www.openssl.org/source/).

### Passaggi successivi {#next-steps-1}

Prima di iniziare a utilizzare Dispatcher, è necessario:

* [Configurare](dispatcher-configuration.md) Dispatcher
* [Configurare AEM](page-invalidate.md) per funzionare con Dispatcher.

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
>
>Le istruzioni per gli ambienti Windows e Unix sono descritte qui.
>
>Fai molta attenzione quando selezioni quale eseguire.

### Sun Java System Web Server / iPlanet: installazione del server web {#sun-java-system-web-server-iplanet-installing-your-web-server}

Per informazioni dettagliate su come installare questi server web, vedi la relativa documentazione:

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server / iPlanet: aggiunta del modulo Dispatcher {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher viene fornito per:

* **Windows**: una DLL (Dynamic Link Library)
* oppure **Unix**: un DSO (Dynamic Shared Object)

L’archivio di installazione contiene i seguenti file, a seconda che sia stato selezionato Windows o Unix:

| File | Descrizione |
|---|---|
| `disp_ns.dll` | Windows: il file DLL di Dispatcher. |
| `dispatcher.so` | Unix: il file DSO di Dispatcher. |
| `dispatcher.so` | Unix: un esempio di collegamento. |
| `obj.conf.disp` | Esempio di file di configurazione per Sun Java System Web Server / iPlanet. |
| `dispatcher.any` | Esempio di file di configurazione per Dispatcher. |
| README | File Readme contenente istruzioni di installazione e informazioni dell’ultimo minuto. Nota: leggere questo file prima di iniziare l’installazione. |
| CHANGES | Il file Changes in cui sono elencati i problemi risolti in questa e nelle precedenti versioni. |

Per aggiungere Dispatcher al server web, fai quanto segue:

1. Posiziona il file Dispatcher nella directory `plugin` del server web:

### Sun Java System Web Server / iPlanet; configurazione per Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

È necessario configurare il server web utilizzando `obj.conf`. Il kit di installazione di Dispatcher contiene un esempio di file di configurazione denominato `obj.conf.disp`.

1. Accedi a `<WEBSERVER_ROOT>/config`.
1. Apri `obj.conf` per la modifica.
1. Copia la riga che inizia con:\
   `Service fn="dispService"`\
   dal file `obj.conf.disp` alla sezione di inizializzazione del file `obj.conf`.

1. Salva le modifiche.
1. Apri `magnus.conf` per la modifica.
1. Copia le due righe che iniziano con:\
   `Init funcs="dispService, dispInit"`\
   e\
   `Init fn="dispInit"`\
   dal file `obj.conf.disp` alla sezione di inizializzazione del file `magnus.conf`.

1. Salva le modifiche.

>[!NOTE]
>
>Le seguenti configurazioni devono essere tutte su una stessa riga e i parametri `$(SERVER_ROOT)` e `$(PRODUCT_SUBDIR)` devono essere sostituiti dai rispettivi valori.

**Init**

La tabella che segue riporta alcuni esempi che possono essere utilizzati; le voci esatte dipendono dal tipo di server web in uso:

**Windows e Unix**

```
...  
Init funcs="dispService,dispInit" fn="load-modules" shlib="$(SERVER\_ROOT)/plugins/dispatcher.so"  
Init fn="dispInit" config="$(PRODUCT\_SUBDIR)/dispatcher.any" loglevel="1" logfile="$(PRODUCT\_SUBDIR)/logs/dispatcher.log"  
keepalivetimeout="60"  
...
```

Dove:

| Parametro | Descrizione |
|--- |--- |
| config | Posizione e nome del file di configurazione `dispatcher.any.` |
| logfile | Posizione e nome del file di registro. |
| loglevel | Livello di registro per la scrittura dei messaggi nel file di registro: <br/>**0** Errori <br/>**1** Avvisi <br/>**2** Informazioni <br/>**3** Debug <br/>**Nota:** si consiglia di impostare il livello di registro su 3 durante l’installazione e il test e su 0 quando l’esecuzione avviene in un ambiente di produzione. |
| keepalivetimeout | Specifica il timeout keep-alive espresso in secondi. A partire dalla versione 4.2.0 di Dispatcher, il valore keep-alive predefinito è 60. Il valore 0 disattiva keep-alive. |

A seconda delle esigenze è possibile definire Dispatcher come servizio per tuoi oggetti. Per configurare Dispatcher per l’intero sito web, modifica l’oggetto predefinito:


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

* [Configurare](dispatcher-configuration.md) Dispatcher
* [Configurare AEM](page-invalidate.md) per funzionare con Dispatcher.
