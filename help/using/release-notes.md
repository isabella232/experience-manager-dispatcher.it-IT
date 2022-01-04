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
source-git-commit: bd03499fae4096fe5642735eb466276f1a179dec
workflow-type: ht
source-wordcount: '941'
ht-degree: 100%

---

# Note sulla versione di AEM Dispatcher {#aem-dispatcher-release-notes}

## Informazioni sulla versione {#release-information}

|  |  |
|--- |--- |
| Prodotti | Adobe Experience Manager (AEM) Dispatcher |
| Versione | 4.3.4 |
| Tipo | Versione secondaria |
| Data | 29 novembre 2021 |
| URL di download | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services (IIS)](release-notes.md#iis)</li></ul> |
| Compatibilità | AEM 6.1 o superiore |

## Requisiti e prerequisiti di sistema {#system-requirements-and-prerequisites}

Per ulteriori informazioni sui requisiti e prerequisiti, visita la pagina [Piattaforme supportate](https://helpx.adobe.com/it/experience-manager/6-4/sites/deploying/using/technical-requirements.html).

Adobe consiglia vivamente di utilizzare l’ultima versione di AEM Dispatcher per usufruire delle funzionalità e delle correzioni di bug più recenti e delle migliori prestazioni possibili.

## Istruzioni di installazione {#installation-instructions}

Per istruzioni dettagliate, vedi [Installazione di Dispatcher](dispatcher-install.md).

## Cronologia delle versioni {#release-history}

### Versione 4.3.4 (29 novembre 2021) {#nov}

**Correzioni di bug**:

* DISP-833 - Le intestazioni X-Forwarded-Host possono contenere un elenco di nomi host separati da virgola
* DISP-835 - DispatcherUseForwardedHost scambia l&#39;intestazione Host se arriva per ultima

**Miglioramenti**:

* DISP-874 - Crea una configurazione del dispatcher per attivare o disattivare l&#39;implementazione di DISP-818 tramite un flag `DispatcherRestrictUncacheableContent`. Il valore predefinito è Disattivato. Quando è disattivato, rimuove le intestazioni di memorizzazione in cache impostate da mod_expires per il contenuto da non memorizzare nella cache. Questo comportamento è diverso da quello rilevato nella versione 4.3.3 (ma è lo stesso delle versioni precedenti alla 4.3.3). Si consiglia di mantenere l&#39;impostazione predefinita Off per `DispatcherRestrictUncacheableContent` in modo che la cache del browser sia più flessibile. Se, durante l’aggiornamento dalla versione 4.3.3 alla versione 4.3.4, desideri mantenere lo stesso comportamento della versione 4.3.3, imposta esplicitamente `DispatcherRestrictUncacheableContent` su On.
* DISP-841 - Dispatcher non rispetta /serverStaleOnError per il codice di risposta 504
* DISP-874 - Creare una configurazione del dispatcher per attivare o disattivare l&#39;implementazione di DISP-818
* DISP-883 - Traccia che mostra la decomposizione della richiesta URL in Dispatcher
* DISP-944 - pre-caricamento degli URL personalizzati

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
* DISP-826 - Supporto di refetch URL con una stringa di query

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
| Linux | i686 (32 bit) | Nessuna | [dispatcher-apache2.4-linux-i686-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.4.tar.gz) |
| Linux | i686 (32 bit) | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.4.tar.gz) |
| Linux | i686 (32 bit) | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.4.tar.gz) |
| Linux | x86_64 (64 bit) | Nessuna | [dispatcher-apache2.4-linux-x86_64-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.4.tar.gz) |
| Linux | x86_64 (64 bit) | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.4.tar.gz) |
| Linux | x86_64 (64 bit) | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.4.tar.gz) |
| macOS | x86_64 (64 bit) | Nessuna | [dispatcher-apache2.4-darwin-x86_64-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.4.tar.gz) |

### IIS {#iis}

| Piattaforma | Architettura | Supporto OpenSSL | Download |
|---|---|---|---|
| Windows | x86 (32 bit) | Nessuna | [dispatcher-iis-windows-x86-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.4.zip) |
| Windows | x86 (32 bit) | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.4.zip) |
| Windows | x86 (32 bit) | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.4.zip) |
| Windows | x64 (64 bit) | Nessuna | [dispatcher-iis-windows-x64-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.4.zip) |
| Windows | x64 (64 bit) | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.4.zip) |
| Windows | x64 (64 bit) | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.4.zip) |
