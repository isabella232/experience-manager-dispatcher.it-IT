---
title: Ottimizzazione delle prestazioni della cache di un sito web
seo-title: Optimizing a Website for Cache Performance
description: Scopri come progettare il sito web per massimizzare i vantaggi del caching.
seo-description: Dispatcher offers a number of built-in mechanisms that you can use to optimize performance. Learn how to design your web site to maximize the benefits of caching.
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
source-git-commit: 2ca816ac0776d72be651b76ff4f45e0c3ed1450f
workflow-type: ht
source-wordcount: '1134'
ht-degree: 100%

---


# Ottimizzazione delle prestazioni della cache di un sito web {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Le versioni di Dispatcher sono indipendenti da AEM. Potresti essere stato reindirizzato a questa pagina se hai seguito un collegamento alla documentazione di Dispatcher incorporato nella documentazione di una versione precedente di AEM.

Dispatcher offre una serie di meccanismi incorporati che si possono utilizzare per ottimizzare le prestazioni. Questa sezione spiega come progettare il sito web per massimizzare i vantaggi del caching.

>[!NOTE]
>
>Può essere utile ricordare che Dispatcher memorizza la cache su un server web standard. Ciò significa che:
>
>* Puoi memorizzare in cache tutto ciò che può essere archiviato come pagina e si può richiedere tramite un URL
>* Non puoi memorizzare altri elementi, ad esempio intestazioni HTTP, cookie, dati di sessione e dati di moduli.

>
>In generale, molte strategie di caching richiedono la selezione di URL validi e non utilizzano questi dati aggiuntivi.

## Utilizzo di una codifica coerente delle pagine {#using-consistent-page-encoding}

Le intestazioni delle richieste HTTP non vengono memorizzate in cache, pertanto possono verificarsi dei problemi, se si memorizzano le informazioni di codifica della pagina nell’intestazione. In questa situazione, quando Dispatcher richiama una pagina dalla cache, per essa viene utilizzata la codifica predefinita del server web. Esistono due modi per evitare questo problema:

* Se utilizzi una sola codifica, accertati che la codifica utilizzata sul server web sia la stessa della codifica predefinita del sito web AEM.
* Utilizza un tag `<META>` nella sezione HTML `head` per impostare la codifica, come nell’esempio seguente:

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## Evita i parametri URL {#avoid-url-parameters}

Se possibile, evita i parametri URL per le pagine che vuoi memorizzare in cache. Ad esempio, se hai una galleria di immagini, il seguente URL non viene mai memorizzato in cache (a meno che Dispatcher non sia [configurato per farlo](dispatcher-configuration.md#main-pars_title_24)):

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

Tuttavia, puoi inserire questi parametri nell’URL della pagina nel modo che segue:

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>Questo URL richiama la stessa pagina e lo stesso modello del file gallery.html. Nella definizione del modello, è possibile specificare quale script esegue il rendering della pagina oppure utilizzare lo stesso script per tutte le pagine.

## Personalizza in base all&#39;URL {#customize-by-url}

Se consenti agli utenti di modificare la dimensione del font (o di effettuare qualunque altra personalizzazione del layout), accertati che le diverse personalizzazioni siano poi riportate nell’URL.

Ad esempio, i cookie non vengono memorizzati in cache, quindi se memorizzi la dimensione del font in un cookie (o in un meccanismo simile), essa non viene mantenuta per la pagina memorizzata in cache. Di conseguenza, Dispatcher restituisce documenti con qualunque dimensione del font, a caso.

L’inclusione della dimensione del font nell’URL come selettore evita questo problema:

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>Per la maggior parte degli aspetti del layout, è inoltre possibile utilizzare fogli di stile e/o script dal lato client. Questi di solito funzionano molto bene con il caching.
>
>Questa funzione è utile anche per la versione stampata, in cui è possibile utilizzare un URL come: ``
>
>`www.myCompany.com/news/main.print.html`
>
>Utilizzando il globbing dello script della definizione del modello, puoi specificare uno script separato che esegue il rendering delle pagine da stampare.

## Annullamento della validità dei file di immagine utilizzati come titoli {#invalidating-image-files-used-as-titles}

Se esegui il rendering dei titoli di pagina, o di altro testo, come immagini, si consiglia di memorizzare i file in modo che vengano eliminati al momento di un aggiornamento del contenuto della pagina:

1. Posiziona il file di immagine nella stessa cartella della pagina.
1. Utilizza il seguente formato di denominazione per il file di immagine:

   `<page file name>.<image file name>`

Ad esempio, puoi memorizzare il titolo della pagina myPage.html nel file myPage.title.gif. Questo file viene eliminato automaticamente, se la pagina viene aggiornata, quindi qualsiasi modifica al titolo della pagina viene automaticamente riportata nella cache.

>[!NOTE]
>
>Il file di immagine non esiste necessariamente come elemento fisico nell’istanza AEM. Puoi utilizzare uno script che crea il file di immagine in modo dinamico. Dispatcher memorizza quindi il file sul server web.

## Annullamento della validità dei file di immagine utilizzati per la navigazione {#invalidating-image-files-used-for-navigation}

Se utilizzi le immagini per le voci di navigazione, il metodo è sostanzialmente lo stesso utilizzato per i titoli, anche se leggermente più complesso. Memorizza tutte le immagini di navigazione con le pagine di destinazione. Se utilizzi due immagini per la modalità normale e attiva, puoi utilizzare i seguenti script:

* Uno script che visualizza la pagina, come normale.
* Uno script che elabora le richieste “.normal” e restituisce l’immagine normale.
* Uno script che elabora le richieste “.active” e restituisce l’immagine attivata.

È importante creare queste immagini con lo stesso handle di denominazione della pagina, per avere la certezza che un aggiornamento del contenuto elimini queste immagini insieme alla pagina.

Per le pagine non modificate, le immagini rimangono comunque nella cache, sebbene le pagine stesse vengano di solito invalidate automaticamente.

## Personalizzazione {#personalization}

Dispatcher non può memorizzare in cache dati personalizzati, pertanto si consiglia di limitare la personalizzazione ai casi strettamente necessari. Ecco il perché:

* Se utilizzi una pagina iniziale liberamente personalizzabile, questa pagina deve essere composta ogni volta che un utente la richiede.
* Se invece offri una scelta tra 10 pagine iniziali diverse, puoi memorizzare in cache ciascuna di esse, migliorando così le prestazioni.

>[!NOTE]
>
>Se personalizzi ogni pagina (ad esempio, inserendo il nome dell’utente nella barra del titolo), non puoi più memorizzarla in cache e ciò può determinare un notevole impatto sulle prestazioni.
>
>Tuttavia, se proprio devi farlo, puoi:
>
>* Utilizzare iFrames per dividere la pagina in modo che una parte sia identica per tutti gli utenti e l’altra parte sia identica per tutte le pagine dell’utente. A questo punto, puoi memorizzare in cache entrambe le parti.
>* Utilizzare JavaScript lato client per visualizzare le informazioni personalizzate. Tuttavia, devi accertarti che la pagina venga comunque visualizzata correttamente anche se un utente disattiva JavaScript.

>


## Connessioni permanenti {#sticky-connections}

[Le connessioni permanenti](dispatcher.md#TheBenefitsofLoadBalancing) garantiscono che di un utente siano composti tutti sullo stesso server. Se un utente esce da questa cartella e successivamente vi rientra, la connessione è ancora attiva. Definisci una cartella in modo che contenga tutti i documenti che richiedono connessioni permanenti per il sito web. Cerca di non avere altri documenti in quella cartella. Ciò impatta sul bilancimento del carico, se utilizzi pagine e dati di sessione personalizzati.

## Tipi MIME {#mime-types}

Esistono due modi in cui un browser può determinare il tipo di file:

1. In base alla sua estensione (ad esempio, .html, .gif, .jpg, ecc.)
1. In base al tipo MIME che il server invia con il file.

Per la maggior parte dei file, il tipo MIME è implicito nell’estensione del file. Ovvero:

1. In base alla sua estensione (ad esempio, .html, .gif, .jpg, ecc.)
1. In base al tipo MIME che il server invia con il file.

Se il nome del file non ha estensione, viene visualizzato come testo normale.

Il tipo MIME fa parte dell’intestazione HTTP e, come tale, Dispatcher non la memorizza in cache. Se l’applicazione AEM restituisce file che non hanno un’estensione di file riconosciuta e il riconoscimento si basa solo sul tipo MIME, questi file potrebbero non essere visualizzati correttamente.

Per avere la certezza i file siano memorizzati in cache correttamente, attieniti alle seguenti linee guida:

* Verifica che i file abbiano sempre l’estensione corretta.
* Evita gli script di server di file generici che hanno URL del tipo download.jsp?file=2214. Riscrivi lo script in modo da utilizzare URL contenenti la specifica del file; nell’esempio precedente, sarebbe download.2214.pdf.

