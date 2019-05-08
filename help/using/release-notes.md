---
title: Note sulla versione di Dispatcher AEM
seo-title: Note sulla versione di Dispatcher AEM
description: Note sulla versione specifiche del dispatcher di Adobe Experience Manager
seo-description: Note sulla versione specifiche del dispatcher di Adobe Experience Manager
uuid: ae 3 ccf 62-0514-4 c 03-a 3 b 9-71799 a 482 cbd
topic-tags: note sulla versione
content-type: riferimento
products: SG_ EXPERIENCEMANAGER/6.4
discoiquuid: ff 3 d 38 e 0-71 c 9-4 b 41-85 f 9-fa 896393 aac 5
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# Note sulla versione di Dispatcher AEM{#aem-dispatcher-release-notes}

## Informazioni sulla versione {#release-information}

|  |
|--- |--- |
| Prodotti | Dispatcher di Adobe Experience Manager (AEM) |
| Versione | 4.3.2 |
| Tipo | Versione secondaria |
| Data | 31 gennaio 2019 |
| URL download | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services (IIS)](release-notes.md#iis)</li></ul> |
| Compatibilità | AEM 6.1 o versione successiva |

## Requisiti di sistema e prerequisiti {#system-requirements-and-prerequisites}

Per [ulteriori informazioni sui requisiti e sui prerequisiti, consultate](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html) la pagina Piattaforme supportate.

Adobe consiglia vivamente di utilizzare la versione più recente di Dispatcher AEM per sfruttare le funzionalità più recenti, le correzioni di bug più recenti e le prestazioni migliori.

## Istruzioni di installazione {#installation-instructions}

Per istruzioni dettagliate, consultate [Installazione del dispatcher](dispatcher-install.md).

## Cronologia rilascio {#release-history}

### Rilascio 4.3.2 (2019-Gen -31) {#jan}

**Correzioni di bug**:

* DISP -734: il dispatcher provoca l&#39;arresto anomalo nel filtro insert_ output_ se non viene impostato come gestore
* DISP -735 - RES non funziona su Alpine Linux
* DISP -740: il caricamento di dispatcher in macos Mojave è disabilitato per impostazione predefinita
* DISP -742: le richieste bloccate possono perdere informazioni per le risorse protette dall&#39;autenticazione

**Miglioramenti**:

* DISP -746: le stringhe non etichettate in dispatcher. any devono generare un avviso

**Nuova funzionalità**:

* DISP -747: fornire informazioni su richiesta nell&#39;ambiente Apache

### Rilascio 4.3.1 (2018-ott -16) {#oct}

**Correzioni di bug**:

* DISP -656: Dispatcher ha errato etag Header
* DISP -694 - Disattivazione degli avvisi quando le connessioni continue vanno a stanti
* DISP -714: la gestione delle sessioni basata su cookie non funziona in IIS
* DISP -715 - Flag protetto per il cookie di rendering
* DISP -720: i file temporanei non chiusi possono causare l&#39;esaurimento (troppi file aperti)
* DISP -721 - Il dispatcher interrompe il sondaggio () quando Apache riavvia il figlio con gentilezza
* DISP -722: i file della cache vengono creati con la modalità ottale 0600
* DISP -723: Timeout di 10 minuti implicito (e ritry) quando il timeout rendering è impostato su 0
* DISP -725: i caratteri finali dopo le stringhe vengono convertiti in modo invisibile in valore senza nome
* DISP -726: registrare un avviso quando nessuna fattoria corrisponde effettivamente all&#39;host in arrivo
* DISP -727: il dispatcher controlla la lunghezza del contenuto della richiesta per i file vuoti della cache
* DISP -730 - 404 quando si tenta di accedere al file dell&#39;intestazione tramite dispatcher
* DISP -731: Dispatcher è vulnerabile all&#39;inserimento di log
* DISP -732: il dispatcher deve rimuovere l&#39;URL consecutivo
* DISP -733: il dispatcher deve impostare (calculate) un&#39;intestazione age

**Miglioramenti**:

* DISP -656: Dispatcher ha errato etag Header
* DISP -694 - Disattivazione degli avvisi quando le connessioni continue vanno a stanti
* DISP -715 - Flag protetto per il cookie di rendering
* DISP -722: i file della cache vengono creati con la modalità ottale 0600
* DISP -726: registrare un avviso quando nessuna fattoria corrisponde effettivamente all&#39;host in arrivo

### Rilascio 4.3.0 (2018-Jun -13) {#jun}

**Correzioni di bug**:

* DISP -682 - Livello di registro numerico non applicato correttamente
* DISP -685 - I binari Solaris a 32 bit hanno un riferimento non definito a__ divdi 3
* DISP -688: il dispatcher non restituisce l&#39;intestazione «X-Cache-Info» su 404 response
* DISP -690: l&#39;intestazione dell&#39;ultima modifica non è memorizzata nella cache
* DISP -691 - Violations Violations in w 3 wp. exe
* DISP -693 - Need to update architectural details for solaris server on dispatcher download page
* DISP -695 - Edizione con dispatcherlog nel modulo Dispatcher 4.2.3
* DISP -698: il TTL di Dispatcher deve supportare le direttive s-maxage e private
* DISP -700 - Il modulo non funziona correttamente su Alpux
* DISP -704: le richieste del browser contenenti % 2 b vengono inviate all&#39;editore non codificato
* DISP -705: arresto anomalo del dispatcher causato dalla doppia libertà o corruzione (in alto)
* DISP -706: durante l&#39;annullamento della validità, dispatcher è un riferimento di riferimento indietro che può causare un ciclo infinito
* DISP -709 - Blocco di alcune estensioni URL personalizzate
* DISP -710 - Build per Linux non utilizzabile su Cent OS 6

**Miglioramenti**:

* DISP -652: il dispatcher fornisce un&#39;intestazione data errata

## Risorse utili {#helpful-resources}

* [Panoramica di Dispatcher AEM](dispatcher.md)

## Download {#downloads}

### Apache 2.4 {#apache}

| Piattaforma | Architettura | Supporto SSL | Scarica |
|---|---|---|---|
| AIX | Powerpc (32 bit) | No | [dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz) |
| AIX | Powerpc (32 bit) | Sì | [dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz) |
| AIX | Powerpc (64 bit) | No | [dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz) |
| AIX | Powerpc (64 bit) | Sì | [dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz) |
| Linux | i 686 (32 bit) | No | [dispatcher-apache2.4-linux-i686-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.2.tar.gz) |
| Linux | i 686 (32 bit) | Sì | [dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz) |
| Linux | x 86_ 64 (64 bit) | No | [dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz) |
| Linux | x 86_ 64 (64 bit) | Sì | [dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz) |
| macOS | x 86_ 64 (64 bit) | No | [dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz) |
| Solaris | AMD (32 bit) | No | [dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz) |
| Solaris | AMD (32 bit) | Sì | [dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz) |
| Solaris | AMD (64 bit) | No | [dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz) |
| Solaris | AMD (64 bit) | Sì | [dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz) |
| Solaris | SPARC (32 bit) | No | [dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz) |
| Solaris | SPARC (32 bit) | Sì | [dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz) |
| Solaris | SPARC (64 bit) | No | [dispatcher-apache2.4-solaris-sparcv9-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-4.3.2.tar.gz) |
| Solaris | SPARC (64 bit) | Sì | [dispatcher-apache2.4-solaris-sparcv9-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-ssl-4.3.2.tar.gz) |

### IIS {#iis}

| Piattaforma | Architettura | Supporto SSL | Scarica |
|---|---|---|---|
| Windows | x 86 (32 bit) | No | [dispatcher-iis-windows-x86-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.2.zip) |
| Windows | x 86 (32 bit) | Sì | [dispatcher-iis-windows-x86-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl-4.3.2.zip) |
| Windows | x 64 (64 bit) | No | [dispatcher-iis-windows-x64-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.2.zip) |
| Windows | x 64 (64 bit) | Sì | [dispatcher-iis-windows-x64-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl-4.3.2.zip) |
