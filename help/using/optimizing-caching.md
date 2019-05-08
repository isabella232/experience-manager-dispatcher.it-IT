---
title: Ottimizzazione di un sito Web per le prestazioni cache
seo-title: Ottimizzazione di un sito Web per le prestazioni cache
description: Scoprite come progettare il sito Web per massimizzare i vantaggi del caching.
seo-description: Il dispatcher offre una serie di meccanismi integrati che potete utilizzare per ottimizzare le prestazioni. Scoprite come progettare il sito Web per massimizzare i vantaggi del caching.
uuid: 2 d 4114 d 1-f 464-4 e 10-b 25 c-a 1 b 9 a 9 c 715 d 1
contentOwner: Utente
products: SG_ EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: riferimento
discoiquuid: ba 323503-1494-4048-941 d-c 1 d 14 f 2 e 85 b 2
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# Ottimizzazione di un sito Web per le prestazioni cache {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Le versioni del dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporata nella documentazione per una versione precedente di AEM.

Il dispatcher offre una serie di meccanismi integrati che potete utilizzare per ottimizzare le prestazioni. Questa sezione indica come progettare il sito Web per massimizzare i vantaggi del caching.

>[!NOTE]
>
>Potrebbe essere utile ricordare che il dispatcher memorizza la cache su un server Web standard. Ciò significa che:
>
>* can cache everything that you can store as a page and request using an URL
>* non può memorizzare altri elementi, ad esempio intestazioni HTTP, cookie, dati sessione e dati del modulo.
>
>
In generale, molte strategie di caching richiedono di selezionare URL buoni e non si basano su tali dati aggiuntivi.

## Utilizzo di codifica pagina coerente {#using-consistent-page-encoding}

Le intestazioni della richiesta HTTP non sono memorizzate nella cache e si possono verificare problemi se si memorizzano le informazioni di codifica pagina nell&#39;intestazione. In questa situazione, quando il dispatcher utilizza una pagina dalla cache, per la pagina viene utilizzata la codifica predefinita del server Web. Esistono due modi per evitare questo problema:

* Se utilizzate una sola codifica, accertatevi che la codifica utilizzata sul server Web sia identica alla codifica predefinita del sito Web AEM.
* Utilizzate un `<META>` tag nella `head` sezione HTML per impostare la codifica, come nell&#39;esempio seguente:

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## Evitare i parametri URL {#avoid-url-parameters}

Se possibile, evitate i parametri URL per le pagine che desiderate memorizzare nella cache. Ad esempio, se si dispone di una galleria di immagini, il seguente URL non viene mai memorizzato nella cache (a meno che non sia [configurato di conseguenza](dispatcher-configuration.md#main-pars_title_24)):

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

Tuttavia, potete inserire questi parametri nell&#39;URL della pagina, come segue:

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>Questo URL richiama la stessa pagina e lo stesso modello di gallery. html. Nella definizione del modello, è possibile specificare quale script esegue il rendering della pagina, oppure è possibile utilizzare lo stesso script per tutte le pagine.

## Personalizza per URL {#customize-by-url}

Se consentite agli utenti di modificare le dimensioni del font (o qualsiasi altra personalizzazione di layout), accertatevi che le diverse personalizzazioni siano presenti nell&#39;URL.

Ad esempio, i cookie non vengono memorizzati nella cache, quindi se archiviate la dimensione del font in un cookie (o meccanismo simile), le dimensioni del font non vengono mantenute per la pagina memorizzata nella cache. Di conseguenza, il dispatcher restituisce documenti di qualsiasi dimensione di font a casuale.

L&#39;inclusione della dimensione font nell&#39;URL come selettore evita il problema:

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>Nella maggior parte dei casi, è possibile utilizzare anche fogli di stile e/o script sul lato client. In genere funzionano bene con il caching.
>
>Questa funzione è utile anche per la versione di stampa, in cui potete usare un URL come: «»
>
>`www.myCompany.com/news/main.print.html`
>
>Utilizzando la descrizione dello script della definizione del modello, è possibile specificare uno script separato per il rendering delle pagine di stampa.

## Annullamento della validità dei file immagine utilizzati come titoli {#invalidating-image-files-used-as-titles}

Se il rendering dei titoli delle pagine o di altro tipo viene eseguito come immagini, si consiglia di archiviare i file in modo che vengano eliminati su un aggiornamento del contenuto sulla pagina:

1. Inserite il file immagine nella stessa cartella della pagina.
1. Usate il formato di denominazione seguente per il file immagine:

   `<page file name>.<image file name>`

Ad esempio, è possibile memorizzare il titolo della pagina myPage.html nel file mypage. title. gif. Questo file viene eliminato automaticamente se la pagina viene aggiornata, quindi qualsiasi modifica al titolo della pagina viene riflessa automaticamente nella cache.

>[!NOTE]
>
>Il file immagine non esiste necessariamente fisicamente sull&#39;istanza AEM. È possibile utilizzare uno script che crea in modo dinamico il file immagine. Dispatcher quindi memorizza il file sul server Web.

## Annullamento della validità dei file immagine utilizzati per la navigazione {#invalidating-image-files-used-for-navigation}

Se utilizzate delle immagini per le voci di navigazione, il metodo è sostanzialmente lo stesso dei titoli, ma è più complesso. Memorizzate tutte le immagini di navigazione con le pagine di destinazione. Se si utilizzano due immagini per normale e attive, è possibile utilizzare i seguenti script:

* Uno script che visualizza la pagina come normale.
* Uno script che elabora le richieste &quot;. normal&quot; e restituisce la foto normale.
* Uno script che elabora &quot;. active&quot; richiede e restituisce l&#39;immagine attivata.

È importante creare queste immagini con la stessa handle di denominazione della pagina, per fare in modo che un aggiornamento del contenuto elimini queste immagini e la pagina.

Per le pagine che non vengono modificate, le immagini rimangono nella cache, anche se in genere le pagine vengono invalidate automaticamente.

## Personalizzazione {#personalization}

Il dispatcher non può memorizzare nella cache i dati personalizzati, pertanto è consigliabile limitare la personalizzazione a dove è necessario. Per spiegare il motivo:

* Se utilizzate una pagina iniziale personalizzabile, tale pagina deve essere composta ogni volta che l&#39;utente la richiede.
* Se, invece, offrirete una scelta di 10 pagine iniziali diverse, potete memorizzarle nella cache a ciascuno di essi, migliorando le prestazioni.

>[!NOTE]
>
>Se personalizzate ogni pagina (ad esempio inserendo il nome dell&#39;utente nella barra del titolo) non potete memorizzarla nella cache, il che può provocare un impatto sulle prestazioni principale.
>
>Tuttavia, in tal caso, potete:
>
>* utilizzare iframe per dividere la pagina in una parte unica per tutti gli utenti e una per tutte le pagine dell&#39;utente. Potete quindi memorizzare entrambe le parti nella cache.
>* utilizzare javascript lato client per visualizzare informazioni personalizzate. Tuttavia, è necessario assicurarsi che la pagina venga visualizzata correttamente se un utente disattiva javascript.
>



## Connessioni fissi {#sticky-connections}

[Le connessioni fisso](dispatcher.md#TheBenefitsofLoadBalancing) garantiscono che i documenti per un utente siano tutti composti sullo stesso server. Se un utente lascia questa cartella e successivamente la restituisce, la connessione continua a essere supportata. Definite una cartella per contenere tutti i documenti che richiedono connessioni fissi per il sito Web. Provate a non avere altri documenti. Questo incide sul bilanciamento del carico se usi pagine personalizzate e dati di sessione.

## Tipi MIME {#mime-types}

Esistono due modi in cui un browser può determinare il tipo di file:

1. Per estensione (ad es.html. gif.jpg ecc.)
1. Per tipo MIME che il server invia insieme al file.

Per la maggior parte dei file, il tipo MIME è implicito nell&#39;estensione del file. es.:

1. Per estensione (ad es.html. gif.jpg ecc.)
1. Per tipo MIME che il server invia insieme al file.

Se il nome del file non ha estensione, viene visualizzato come testo normale.

Il tipo MIME fa parte dell&#39;intestazione HTTP e, come tale, il dispatcher non la cache. Se l&#39;applicazione AEM restituisce file che non hanno un file riconosciuto, ma si fa affidamento sul tipo MIME, questi file potrebbero essere visualizzati in modo errato.

Per verificare che i file siano memorizzati nella cache correttamente, attenetevi alle seguenti linee guida:

* Verificate che i file abbiano sempre l&#39;estensione corretta.
* Evitare di utilizzare script di servizi generici, con URL quali download. jsp? file = 2214. Riscrivere lo script per utilizzare gli URL contenenti la specifica file; per l&#39;esempio precedente viene scaricato .2214. pdf.

