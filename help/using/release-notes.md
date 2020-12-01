---
title: Note sulla versione di AEM Dispatcher
seo-title: Note sulla versione di AEM Dispatcher
description: Note sulla versione specifiche di Adobe Experience Manager Dispatcher
seo-description: Note sulla versione specifiche di Adobe Experience Manager Dispatcher
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
translation-type: tm+mt
source-git-commit: 328bc82673783b4a2df2d68481fa7eec88b74b01
workflow-type: tm+mt
source-wordcount: '805'
ht-degree: 8%

---


# Note sulla versione di AEM Dispatcher{#aem-dispatcher-release-notes}

## Informazioni sulla versione {#release-information}

|  |  |
|--- |--- |
| Prodotti | Dispatcher Adobe Experience Manager (AEM) |
| Versione | 4.3.3 |
| Tipo | Rilascio secondario |
| Data | 18 ottobre 2019 |
| URL di download | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services (IIS)](release-notes.md#iis)</li></ul> |
| Compatibilità | AEM 6.1 o superiore |

## Requisiti di sistema e prerequisiti {#system-requirements-and-prerequisites}

Per ulteriori informazioni sui requisiti e i prerequisiti, vedere la pagina [Piattaforme supportate](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html).

 Adobe consiglia vivamente di utilizzare la versione più recente di AEM Dispatcher per utilizzare le funzionalità più recenti, le correzioni di bug più recenti e le prestazioni migliori.

## Istruzioni di installazione {#installation-instructions}

Per istruzioni dettagliate, vedere [Installazione del dispatcher](dispatcher-install.md).

## Cronologia rilascio {#release-history}

### Release 4.3.3 (2019-ott-18) {#october}

**Correzioni di bug**:

* DISP-739 - dispatcher LogLevel: **il livello** non funziona
* DISP-749 - Arresto anomalo del dispatcher Linux alpino con livello di log traccia

**Miglioramenti**:

* DISP-813 - Supporto in Dispatcher per openssl 1.1.x
* DISP-814 - Errori Apache 40x durante i download della cache
* DISP-818 - mod_expires aggiunge intestazioni Cache-Control per contenuti non memorizzabili nella cache
* DISP-821: non memorizzare il contesto del registro nel socket
* DISP-822: il dispatcher deve utilizzare ppoll invece di selezionare
* DISP-824 - Secure DispatcherUseForwardedHost
* DISP-825 - Messaggio speciale di registro in assenza di spazio su disco
* DISP-826 - Supporto di URI di recupero con una stringa di query

**Nuove funzioni**:

* DISP-703 - Rapporto di hit cache specifico per azienda
* DISP-827 - Server locale per test
* DISP-828: creazione dell&#39;immagine docker di test per il dispatcher

### Release 4.3.2 (2019-Jan-31) {#jan}

**Correzioni di bug**:

* DISP-734 - Il dispatcher causa l&#39;arresto anomalo in insert_output_filter se non viene impostato come gestore
* DISP-735 - Gli RE non funzionano su Linux alpino
* DISP-740 - Il dispatcher di caricamento in macOS Mojave è disattivato per impostazione predefinita
* DISP-742: le richieste bloccate possono causare la perdita di informazioni sulle risorse protette del controllo di autenticazione

**Miglioramenti**:

* DISP-746 - Stringhe senza etichetta nel dispatcher.any deve generare un avviso

**Nuova funzione**:

* DISP-747 - Fornisce informazioni sulle richieste in ambiente Apache

### Release 4.3.1 (2018-ott-16) {#oct}

**Correzioni di bug**:

* DISP-656 - Il dispatcher utilizza un&#39;intestazione ETag errata
* DISP-694 - Sopprimere gli avvertimenti quando mantenere in vita le connessioni diventano stantio
* DISP-714: la gestione delle sessioni basata su cookie non funziona in IIS
* DISP-715 - Flag di protezione per il cookie renderid
* DISP-720 - I file temporanei non chiusi possono causare esaurimento (troppi file aperti)
* DISP-721 - Il dispatcher interrompe il sondaggio() quando Apache riavvia correttamente l&#39;elemento secondario
* DISP-722: i file cache vengono creati con la modalità ottale 0600
* DISP-723: timeout implicito di 10 minuti (e riprovare) quando i timeout di rendering sono impostati su 0
* DISP-725 - Caratteri finali dopo la conversione delle stringhe in valori senza nome
* DISP-726 - Registrare un avviso quando nessuna farm corrisponde effettivamente all&#39;host in ingresso
* DISP-727 - Il dispatcher verifica la lunghezza del contenuto richiesta per i file cache vuoti
* DISP-730 - 404 quando si tenta di accedere al file di intestazione tramite dispatcher
* DISP-731: il dispatcher è vulnerabile all&#39;iniezione di log
* DISP-732: il dispatcher deve rimuovere ‘/’ consecutivo nell&#39;URL
* DISP-733: il dispatcher deve impostare (calcolare) un&#39;intestazione età

**Miglioramenti**:

* DISP-656 - Il dispatcher utilizza un&#39;intestazione ETag errata
* DISP-694 - Sopprimere gli avvertimenti quando mantenere in vita le connessioni diventano stantio
* DISP-715 - Flag di protezione per il cookie renderid
* DISP-722: i file cache vengono creati con la modalità ottale 0600
* DISP-726 - Registrare un avviso quando nessuna farm corrisponde effettivamente all&#39;host in ingresso

### Release 4.3.0 (2018-giu-13) {#jun}

**Correzioni di bug**:

* DISP-682 - Livello di registro numerico non applicato correttamente
* DISP-685 - I binari Solaris SPARC a 32 bit hanno un riferimento indefinito a __divdi3
* DISP-688 - Il dispatcher non restituisce l&#39;intestazione &quot;X-Cache-Info&quot; nella risposta 404
* DISP-690: l&#39;ultima intestazione modificata non è memorizzabile nella cache
* DISP-691 - Violazioni di accesso in w3wp.exe
* DISP-693 - È necessario aggiornare i dettagli architettonici per i server solaris nella pagina di download del dispatcher
* DISP-695 - Problema con il livello DispatcherLog nel modulo Dispatcher 4.2.3
* DISP-698 - Dispatcher TTL deve supportare le direttive s-maxage e private
* DISP-700 - Il modulo non funziona correttamente su Alpine Linux
* DISP-704: le richieste browser contenenti %2b vengono inviate al server di pubblicazione senza codifica
* DISP-705: arresto anomalo del dispatcher a causa di doppia assenza o danneggiamento (fasttop)
* DISP-706 - Durante l&#39;annullamento della validità, il dispatcher segue i simboli di riferimento posteriori che possono causare un ciclo infinito
* DISP-709 - Blocco di alcune estensioni URL vuote
* DISP-710 - Costruzioni per Linux non utilizzabili su Cent OS 6

**Miglioramenti**:

* DISP-652 - Il dispatcher visualizza l&#39;intestazione errata della data

## Risorse utili {#helpful-resources}

* [Panoramica del dispatcher AEM](dispatcher.md)

## Download {#downloads}

### Apache 2.4 {#apache}

| Platform | Architettura | Supporto OpenSSL | Scarica |
|---|---|---|---|
| Linux | i686 (32 bit) | Nessuno | [dispatcher-apache2.4-linux-i686-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.3.tar.gz) |
| Linux | i686 (32 bit) | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz) |
| Linux | i686 (32 bit) | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz) |
| Linux | x86_64 (64 bit) | Nessuno | [dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz) |
| Linux | x86_64 (64 bit) | 1,0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz) |
| Linux | x86_64 (64 bit) | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz) |
| macOS | x86_64 (64 bit) | Nessuno | [dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz) |

### IIS {#iis}

| Piattaforma | Architettura | Supporto OpenSSL | Scarica |
|---|---|---|---|
| Windows | x86 (32 bit) | Nessuno | [dispatcher-iis-windows-x86-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.3.zip) |
| Windows | x86 (32 bit) | 1,0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip) |
| Windows | x86 (32 bit) | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip) |
| Windows | x64 (64 bit) | Nessuno | [dispatcher-iis-windows-x64-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.3.zip) |
| Windows | x64 (64 bit) | 1,0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip) |
| Windows | x64 (64 bit) | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip) |
