---
title: Ottimizzazione di un sito Web per le prestazioni della cache
seo-title: Ottimizzazione di un sito Web per le prestazioni della cache
description: Scoprite come progettare il sito Web per massimizzare i vantaggi della memorizzazione nella cache.
seo-description: Dispatcher offre una serie di meccanismi incorporati che è possibile utilizzare per ottimizzare le prestazioni. Scoprite come progettare il sito Web per massimizzare i vantaggi della memorizzazione nella cache.
uuid: 2d4114d1-f464-4e10-b25c-a1b9a9c715d1
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: ba323503-1494-4048-941d-c1d14f2e85b2
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 2ca816ac0776d72be651b76ff4f45e0c3ed1450f
workflow-type: tm+mt
source-wordcount: '1167'
ht-degree: 3%

---


# Ottimizzazione di un sito Web per le prestazioni della cache {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

Dispatcher offre una serie di meccanismi incorporati che è possibile utilizzare per ottimizzare le prestazioni. In questa sezione viene illustrato come progettare il sito Web per massimizzare i vantaggi della memorizzazione nella cache.

>[!NOTE]
>
>Potrebbe essere utile ricordare che il Dispatcher memorizza la cache su un server Web standard. Ciò significa che:
>
>* è possibile memorizzare nella cache tutto ciò che è possibile memorizzare come pagina e richiederlo utilizzando un URL
>* non è in grado di memorizzare altri elementi, ad esempio intestazioni HTTP, cookie, dati di sessione e dati del modulo.

>
>
In generale, molte strategie di caching richiedono la selezione di buoni URL e non affidarsi a questi dati aggiuntivi.

## Utilizzo della codifica di pagina coerente {#using-consistent-page-encoding}

Le intestazioni delle richieste HTTP non sono memorizzate nella cache, pertanto si possono verificare problemi se si memorizzano le informazioni di codifica delle pagine nell’intestazione. In questa situazione, quando Dispatcher invia una pagina dalla cache, per la pagina viene utilizzata la codifica predefinita del server Web. Esistono due modi per evitare questo problema:

* Se utilizzate una sola codifica, accertatevi che la codifica utilizzata sul server Web sia la stessa della codifica predefinita del sito Web AEM.
* Utilizzate un tag `<META>` nella sezione HTML `head` per impostare la codifica, come nell&#39;esempio seguente:

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## Evitare i parametri URL {#avoid-url-parameters}

Se possibile, evitate i parametri URL per le pagine che desiderate memorizzare nella cache. Ad esempio, se disponete di una raccolta immagini, l&#39;URL seguente non viene mai memorizzato nella cache (a meno che Dispatcher non sia [configurato di conseguenza](dispatcher-configuration.md#main-pars_title_24)):

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

Tuttavia, potete inserire questi parametri nell’URL della pagina, come segue:

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>Questo URL richiama la stessa pagina e lo stesso modello di galleria.html. Nella definizione del modello, è possibile specificare quale script esegue il rendering della pagina oppure utilizzare lo stesso script per tutte le pagine.

## Personalizza per URL {#customize-by-url}

Se consentite agli utenti di modificare la dimensione del font (o qualsiasi altra personalizzazione del layout), accertatevi che le diverse personalizzazioni siano riportate nell’URL.

Ad esempio, i cookie non sono memorizzati nella cache, pertanto se si memorizzano le dimensioni del font in un cookie (o in un meccanismo simile), la dimensione del font non viene mantenuta per la pagina memorizzata nella cache. Di conseguenza, il dispatcher restituisce i documenti di qualsiasi dimensione di font a caso.

L&#39;inclusione della dimensione del font nell&#39;URL come selettore evita questo problema:

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>Per la maggior parte degli aspetti del layout, è anche possibile utilizzare fogli di stile e/o script sul lato client. Questi di solito funzionano molto bene con la cache.
>
>Questa funzione è utile anche per una versione di stampa in cui potete usare un URL come: &quot;
>
>`www.myCompany.com/news/main.print.html`
>
>Utilizzando la combinazione di script della definizione del modello, potete specificare uno script separato che esegue il rendering delle pagine di stampa.

## Annullamento della convalida dei file immagine utilizzati come titoli {#invalidating-image-files-used-as-titles}

Se eseguite il rendering dei titoli di pagina o di altro testo come immagini, si consiglia di memorizzare i file in modo che vengano eliminati in seguito a un aggiornamento del contenuto sulla pagina:

1. Inserite il file immagine nella stessa cartella della pagina.
1. Usate il seguente formato di denominazione per il file immagine:

   `<page file name>.<image file name>`

Ad esempio, è possibile memorizzare il titolo della pagina myPage.html nel file myPage.title.gif. Questo file viene eliminato automaticamente se la pagina viene aggiornata, pertanto qualsiasi modifica al titolo della pagina viene automaticamente riflessa nella cache.

>[!NOTE]
>
>Il file immagine non esiste necessariamente fisicamente nell&#39;istanza AEM. È possibile utilizzare uno script che crea in modo dinamico il file immagine. Il dispatcher memorizza quindi il file sul server Web.

## Annullamento della convalida dei file immagine utilizzati per la navigazione {#invalidating-image-files-used-for-navigation}

Se si utilizzano le immagini per le voci di navigazione, il metodo è sostanzialmente lo stesso che con i titoli, leggermente più complessi. Archiviate tutte le immagini di navigazione con le pagine di destinazione. Se si utilizzano due immagini per la modalità normale e attiva, è possibile utilizzare i seguenti script:

* Uno script che visualizza la pagina come normale.
* Uno script che elabora richieste &quot;.normal&quot; e restituisce l&#39;immagine normale.
* Uno script che elabora &quot;.active&quot; richiede e restituisce l&#39;immagine attivata.

È importante creare queste immagini con lo stesso handle di denominazione della pagina, per fare in modo che un aggiornamento del contenuto elimini sia queste immagini che la pagina.

Per le pagine che non vengono modificate, le immagini rimangono nella cache, anche se le pagine stesse vengono in genere invalidate automaticamente.

## Personalizzazione {#personalization}

Il Dispatcher non è in grado di memorizzare nella cache i dati personalizzati, pertanto si consiglia di limitare la personalizzazione a dove necessario. Per illustrare i motivi:

* Se utilizzate una pagina iniziale liberamente personalizzabile, tale pagina deve essere composta ogni volta che un utente la richiede.
* Se, invece, si offre una scelta di 10 pagine iniziali diverse, è possibile memorizzare nella cache ciascuna di esse, migliorando così le prestazioni.

>[!NOTE]
>
>Se personalizzate ciascuna pagina (ad esempio inserendo il nome dell’utente nella barra del titolo) non potete memorizzarla nella cache, il che può avere un impatto maggiore sulle prestazioni.
>
>Tuttavia, se dovete eseguire questa operazione, potete:
>
>* utilizzate iFrame per dividere la pagina in una parte identica per tutti gli utenti e in una parte identica per tutte le pagine dell&#39;utente. Potete quindi memorizzare nella cache entrambe le parti.
>* utilizzate JavaScript lato client per visualizzare informazioni personalizzate. Tuttavia, è necessario verificare che la pagina venga visualizzata correttamente anche se l&#39;utente disattiva JavaScript.

>



## Connessioni permanenti {#sticky-connections}

[Connessione ](dispatcher.md#TheBenefitsofLoadBalancing) fissa, assicurarsi che i documenti per un utente siano tutti composti sullo stesso server. Se un utente lascia la cartella e successivamente vi ritorna, la connessione rimane fissa. Definite una cartella per contenere tutti i documenti che richiedono connessioni permanenti per il sito Web. Cercate di non inserire altri documenti. Questo influisce sul bilanciamento del carico se utilizzate pagine personalizzate e dati di sessione.

## Tipi MIME {#mime-types}

Esistono due modi in cui un browser può determinare il tipo di file:

1. Per estensione (ad esempio .html, .gif, .jpg, ecc.)
1. Per tipo MIME inviato dal server con il file.

Per la maggior parte dei file, il tipo MIME è implicito nell&#39;estensione del file. i.e.:

1. Per estensione (ad esempio .html, .gif, .jpg, ecc.)
1. Per tipo MIME inviato dal server con il file.

Se il nome del file non ha estensione, viene visualizzato come testo normale.

Il tipo MIME fa parte dell&#39;intestazione HTTP e, come tale, il dispatcher non la memorizza nella cache. Se l&#39;applicazione AEM restituisce file che non hanno una fine riconosciuta ma che fanno affidamento sul tipo MIME, questi file potrebbero essere visualizzati in modo non corretto.

Per essere certi che i file siano memorizzati nella cache in modo corretto, attenetevi alle seguenti linee guida:

* Accertatevi che i file abbiano sempre l&#39;estensione corretta.
* Evitate gli script di server di file generici, con URL quali download.jsp?file=2214. riscrivere lo script per utilizzare gli URL contenenti la specifica del file; nell’esempio precedente, questo sarà download.2214.pdf.

