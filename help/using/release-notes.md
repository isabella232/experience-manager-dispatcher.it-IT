---
title: Note sulla versione di AEM Dispatcher
seo-title: Note sulla versione di AEM Dispatcher
description: Note sulla versione specifiche di Adobe Experience Manager Dispatcher
seo-description: Note sulla versione specifiche di Adobe Experience Manager Dispatcher
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: note sulla versione
content-type: riferimento
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
translation-type: tm+mt
source-git-commit: 2d72839d459973cba40f6a938ee157198c7cf50a

---


# Note sulla versione di AEM Dispatcher{#aem-dispatcher-release-notes}

## Informazioni sulla versione {#release-information}

|  |  |
|--- |--- |
| Prodotti | Dispatcher di Adobe Experience Manager (AEM) |
| Versione | 4.3.2 |
| Tipo | Minor Release |
| Data | January 31, 2019 |
| Scarica URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services (IIS)](release-notes.md#iis)</li></ul> |
| Compatibilità | AEM 6.1 or higher |

## System requirements and prerequisites {#system-requirements-and-prerequisites}

Please see the Supported Platforms page for more information regarding requirements and prerequisites.[](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html)

Adobe strongly recommends using the latest version of AEM Dispatcher to avail the latest functionality, the most recent bug fixes, and the best possible performance.

## Installation instructions {#installation-instructions}

For detailed instructions, see Installing Dispatcher.[](dispatcher-install.md)

## Release History {#release-history}

### Release 4.3.2 (2019-Jan-31) {#jan}

**Bug Fixes:**

* DISP-734 - Dispatcher causes crash in insert_output_filter if not set as handler
* DISP-735 - Gli RE non funzionano su Linux alpino
* DISP-740 - Il dispatcher di caricamento in macOS Mojave è disattivato per impostazione predefinita
* DISP-742 - Blocked requests may leak information to auth checker protected resources

**Miglioramenti**:

* DISP-746 - Stringhe senza etichetta nel dispatcher.any deve generare un avviso

**Nuova funzione**:

* DISP-747 - Fornisce informazioni sulle richieste in ambiente Apache

### Release 4.3.1 (2018-16 ott-2016) {#oct}

**Correzioni dei** bug:

* DISP-656 - Il dispatcher utilizza un'intestazione ETag errata
* DISP-694 - Sopprimere gli avvertimenti quando mantenere in vita le connessioni diventano stantio
* DISP-714: la gestione delle sessioni basata su cookie non funziona in IIS
* DISP-715 - Flag di protezione per il cookie renderid
* DISP-720 - I file temporanei non chiusi possono causare esaurimento (troppi file aperti)
* DISP-721 - Il dispatcher interrompe il sondaggio() quando Apache riavvia correttamente l'elemento secondario
* DISP-722: i file cache vengono creati con la modalità ottale 0600
* DISP-723: timeout implicito di 10 minuti (e riprovare) quando i timeout di rendering sono impostati su 0
* DISP-725 - Caratteri finali dopo la conversione delle stringhe in valori senza nome
* DISP-726 - Log a warning when no farm actually matches the incoming host
* DISP-727 - Il dispatcher verifica la lunghezza del contenuto richiesta per i file cache vuoti
* DISP-730 - 404 quando si tenta di accedere al file di intestazione tramite dispatcher
* DISP-731 - Dispatcher is vulnerable to Log Injection
* DISP-732: il dispatcher deve rimuovere ‘/’ consecutivo nell'URL
* DISP-733 - Dispatcher should set (calculate) an Age Header

**Improvements:**

* DISP-656 - Dispatcher serves wrong ETag Header
* DISP-694 - Suppress warnings when keep alive connections go stale
* DISP-715 - Flag di protezione per il cookie renderid
* DISP-722: i file cache vengono creati con la modalità ottale 0600
* DISP-726 - Registrare un avviso quando nessuna farm corrisponde effettivamente all'host in ingresso

### Release 4.3.0 (2018-Giu-13) {#jun}

**Correzioni dei** bug:

* DISP-682 - Livello di registro numerico non applicato correttamente
* DISP-685 - I binari Solaris SPARC a 32 bit hanno un riferimento indefinito a __divdi3
* DISP-688 - Il dispatcher non restituisce l'intestazione "X-Cache-Info" nella risposta 404
* DISP-690 - Last-Modified header is not cacheable
* DISP-691 - Access Violations in w3wp.exe
* DISP-693 - Need to update Architectural details for solaris servers on dispatcher download page
* DISP-695 - Problema con il livello DispatcherLog nel modulo Dispatcher 4.2.3
* DISP-698 - Dispatcher TTL needs to support s-maxage and private directives
* DISP-700 - Il modulo non funziona correttamente su Alpine Linux
* DISP-704: le richieste browser contenenti %2b vengono inviate al server di pubblicazione senza codifica
* DISP-705: arresto anomalo del dispatcher a causa di doppia assenza o danneggiamento (fasttop)
* DISP-706 - During invalidation, dispatcher is following back reference symlinks which can cause an infinite loop
* DISP-709 - Blocco di alcune estensioni URL vuote
* DISP-710 - Costruzioni per Linux non utilizzabili su Cent OS 6

**Miglioramenti**:

* DISP-652 - Il dispatcher visualizza l'intestazione errata della data

## Risorse utili {#helpful-resources}

* [Panoramica del dispatcher AEM](dispatcher.md)

## Download {#downloads}

### Apache 2.4 {#apache}

| Piattaforma | Architettura | Supporto SSL | Scarica |
|---|---|---|---|
| AIX | PowerPC (32 bit) | No | [dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz) |
| AIX | PowerPC (32 bit) | Sì | [dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz) |
| AIX | PowerPC (64 bit) | No | [dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz) |
| AIX | PowerPC (64 bit) | Sì | [dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz) |
| Linux | i686 (32 bit) | No | [dispatcher-apache2.4-linux-i686-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.2.tar.gz) |
| Linux | i686 (32 bit) | Sì | [dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz) |
| Linux | x86_64 (64 bit) | No | [dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz) |
| Linux | x86_64 (64 bit) | Sì | [dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz) |
| macOS | x86_64 (64 bit) | No | [dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz) |
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
| Windows | x86 (32 bit) | No | [dispatcher-iis-windows-x86-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.2.zip) |
| Windows | x86 (32 bit) | Sì | [dispatcher-iis-windows-x86-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl-4.3.2.zip) |
| Windows | x64 (64 bit) | No | [dispatcher-iis-windows-x64-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.2.zip) |
| Windows | x64 (64 bit) | Sì | [dispatcher-iis-windows-x64-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl-4.3.2.zip) |
