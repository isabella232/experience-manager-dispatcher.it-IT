---
title: Note sulla versione di AEM Dispatcher
seo-title: AEM Dispatcher Release Notes
description: Note sulla versione specifiche di Adobe Experience Manager Dispatcher
seo-description: Release notes specific to Adobe Experience Manager Dispatcher
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
exl-id: b55c7a34-d57b-4d45-bd83-29890f1524de
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '793'
ht-degree: 100%

---

# Note sulla versione di AEM Dispatcher{#aem-dispatcher-release-notes}

## Informazioni sulla versione {#release-information}

|  |  |
|--- |--- |
| Prodotti | Adobe Experience Manager (AEM) Dispatcher |
| Versione | 4.3.3 |
| Tipo | Versione secondaria |
| Data | 18 ottobre 2019 |
| URL di download | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services (IIS)](release-notes.md#iis)</li></ul> |
| Compatibilità | AEM 6.1 o superiore |

## Requisiti e prerequisiti di sistema {#system-requirements-and-prerequisites}

Per ulteriori informazioni sui requisiti e prerequisiti, visita la pagina [Piattaforme supportate](https://helpx.adobe.com/it/experience-manager/6-4/sites/deploying/using/technical-requirements.html).

Adobe consiglia vivamente di utilizzare l’ultima versione di AEM Dispatcher per usufruire delle funzionalità e delle correzioni di bug più recenti e delle migliori prestazioni possibili.

## Istruzioni di installazione {#installation-instructions}

Per istruzioni dettagliate, vedi [Installazione di Dispatcher](dispatcher-install.md).

## Cronologia delle versioni {#release-history}

### Versione 4.3.3 (18 ottobre 2019) {#october}

**Correzioni di bug**:

* DISP-739 - LogLevel Dispatcher: **il livello** non funziona
* DISP-749 - Arresto anomalo di Dispatcher in Alpine Linux con livello del registro di traccia

**Miglioramenti**:

* DISP-813 - Supporto in Dispatcher per openssl 1.1.x
* DISP-814 - Errori Apache 40x durante i flushing della cache
* DISP-818 - mod_expires aggiunge intestazioni Cache-Control per contenuto non memorizzabile in cache
* DISP-821 - Non archiviare il contesto di registro nel socket
* DISP-822 - Dispatcher deve utilizzare ppoll invece di pselect
* DISP-824 - DispatcherUseForwardedHost sicuro
* DISP-825 - Inserisce nel registro un messaggio speciale quando non c’è più spazio sul disco
* DISP-826 - Supporto di refetch URI con una stringa di query

**Nuove funzioni**:

* DISP-703 - Rapporto hit della cache specifico per farm
* DISP-827 - Server locale per test
* DISP-828 - Crea un’immagine docker di test per Dispatcher

### Versione 4.3.2 (31 gennaio 2019) {#jan}

**Correzioni di bug**:

* DISP-734 - Dispatcher causa un arresto anomalo in insert_output_filter se non viene impostato come handler
* DISP-735 - I RE non funzionano su Alpine Linux
* DISP-740 - Il caricamento di Dispatcher in macOS Mojave è disabilitato per impostazione predefinita
* DISP-742 - Le richieste bloccate potrebbero causare la perdita di informazioni alle risorse protette da controllo di authoring

**Miglioramenti**:

* DISP-746 - Le stringhe non etichettate in dispatcher.any devono generare un avviso

**Nuova funzione**:

* DISP-747 - Fornisce informazioni sulle richieste in ambiente Apache

### Versione 4.3.1 (16 ottobre 2018) {#oct}

**Correzioni di bug**:

* DISP-656 - Dispatcher fornisce un’intestazione ETag sbagliata
* DISP-694 - Elimina gli avvisi quando le connessioni keep alive diventano obsolete
* DISP-714 - La gestione delle sessioni basata su cookie non funziona in IIS
* DISP-715 - Flag sicuro per il cookie renderid
* DISP-720 - I file temporanei non chiusi possono causare la saturazione (troppi file aperti)
* DISP-721 - Dispatcher interrompe il poll() quando Apache effettua un graceful restart dell’elemento figlio
* DISP-722 - I file cache vengono creati con modalità ottale 0600
* DISP-723 - Timeout implicito di 10 minuti (e nuovo tentativo) quando i timeout del rendering sono impostati su 0
* DISP-725 - I caratteri finali dopo le stringhe vengono automaticamente convertiti in valori non denominati
* DISP-726 - Inserisce un avviso nel registro quando nessuna farm corrisponde effettivamente all’host in ingresso
* DISP-727 - Dispatcher controlla la lunghezza del contenuto della richiesta per i file di cache vuoti
* DISP-730 - Errore 404 quando si tenta di accedere al file di intestazione tramite Dispatcher
* DISP-731 - Dispatcher è vulnerabile agli attacchi Log Injection
* DISP-732 - Dispatcher deve rimuovere ‘/’ consecutivi nell’URL
* DISP-733 - Dispatcher deve impostare (calcolare) un’intestazione Age

**Miglioramenti**:

* DISP-656 - Dispatcher fornisce un’intestazione ETag sbagliata
* DISP-694 - Elimina gli avvisi quando le connessioni keep alive diventano obsolete
* DISP-715 - Flag sicuro per il cookie renderid
* DISP-722 - I file cache vengono creati con modalità ottale 0600
* DISP-726 - Inserisce un avviso nel registro quando nessuna farm corrisponde effettivamente all’host in ingresso

### Versione 4.3.0 (13 giugno 2018) {#jun}

**Correzioni di bug**:

* DISP-682 - Livello di registro numerico applicato in modo errato
* DISP-685 - I valori binari di Solaris SPARC a 32 bit hanno un riferimento indefinito a __divdi3
* DISP-688 - Dispatcher non restituisce l’intestazione “X-Cache-Info” nella risposta 404
* DISP-690 - L’intestazione Last-Modified non è memorizzabile in cache
* DISP-691 - Violazioni di accesso in w3wp.exe
* DISP-693 - È necessario aggiornare i dettagli di architettura dei server Solaris nella pagina di download di Dispatcher
* DISP-695 - Problema con il livello DispatcherLog nel modulo Dispatcher 4.2.3
* DISP-698 - Il TTL di Dispatcher deve supportare le direttive s-maxage e private
* DISP-700 - Il modulo non funziona correttamente su Alpine Linux
* DISP-704 - Le richieste del browser contenenti %2b vengono inviate al server di publishing senza codifica
* DISP-705 - Arresto anomalo di Dispatcher a causa di una vulnerabilità DF (Double Free) o di danneggiamento (fasttop)
* DISP-706 - Durante l’annullamento della validità, Dispatcher sta seguendo dei back reference symlink che possono causare un loop infinito
* DISP-709 - Blocca alcune estensioni di URL personalizzati
* DISP-710 - Build per Linux non utilizzabili su Cent OS 6

**Miglioramenti**:

* DISP-652 - L’intestazione della data fornita da Dispatcher non è corretta

## Risorse utili {#helpful-resources}

* [Panoramica di AEM Dispatcher](dispatcher.md)

## Download {#downloads}

### Apache 2.4 {#apache}

| Piattaforma | Architettura | Supporto OpenSSL | Download |
|---|---|---|---|
| Linux | i686 (32 bit) | Nessuna | [dispatcher-apache2.4-linux-i686-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.3.tar.gz) |
| Linux | i686 (32 bit) | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz) |
| Linux | i686 (32 bit) | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz) |
| Linux | x86_64 (64 bit) | Nessuna | [dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz) |
| Linux | x86_64 (64 bit) | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz) |
| Linux | x86_64 (64 bit) | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz) |
| macOS | x86_64 (64 bit) | Nessuna | [dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz) |

### IIS {#iis}

| Piattaforma | Architettura | Supporto OpenSSL | Download |
|---|---|---|---|
| Windows | x86 (32 bit) | Nessuna | [dispatcher-iis-windows-x86-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.3.zip) |
| Windows | x86 (32 bit) | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip) |
| Windows | x86 (32 bit) | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip) |
| Windows | x64 (64 bit) | Nessuna | [dispatcher-iis-windows-x64-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.3.zip) |
| Windows | x64 (64 bit) | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip) |
| Windows | x64 (64 bit) | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip) |
