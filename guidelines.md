---
source-git-commit: 053f90d9a0574c7d470cec6794498919e3340713
translation-type: tm+mt

---
# Linee guida per il contributo alla documentazione di Adobe Experience Manager

## Filosofia della documentazione

Sappiamo che gli utenti di Adobe Experience Manager lavorano in ambienti altamente competitivi, cercando di creare esperienze digitali che li distinguano dalla concorrenza. Pertanto, è fondamentale che quando Adobe offre nuovi strumenti avanzati in AEM, questi strumenti siano integrati da una documentazione accurata e chiara che consenta al cliente di sfruttare immediatamente il proprio investimento AEM e massimizzare il ROI.

L’obiettivo della documentazione AEM è di mettere la documentazione nelle mani degli utenti AEM il prima possibile. Pertanto, diamo priorità a una documentazione accurata e utilizzabile e ci impegniamo a aggiornarla e a migliorarla continuamente.

## Contributi alla documentazione

Al fine di migliorare costantemente la documentazione di AEM, l’intera community di utenti AEM è invitata a contribuire alla documentazione. Sia attraverso richieste pull o problemi, i miglioramenti alla documentazione possono essere correzioni, chiarimenti, espansioni ed esempi aggiuntivi.

## Standard di documentazione

Anche se i contributi alla documentazione sono graditi, qualsiasi contributo alla documentazione di AEM, sotto forma di richiesta pull o di problema, deve essere conforme ai nostri standard di contributi e documentazione.

I contributi che non soddisfano tali standard possono essere rifiutati.

### Documentiamo i casi di utilizzo standard.

La documentazione di AEM copre i casi di utilizzo standard. I casi di utilizzo che esulano dall’ambito di installazione e utilizzo standard del prodotto non fanno parte della documentazione di AEM.

### In genere non documentiamo i bug o le loro soluzioni.

La documentazione di AEM copre i casi di utilizzo standard. Per questo motivo, i bug, gli effetti causati da bug e le soluzioni alternative per i bug non sono generalmente documentati.

Le eccezioni a questa regola si applicano alle note sulla versione in cui possono essere elencati i problemi noti con possibili soluzioni approvate dalla Gestione prodotti AEM.

### I contributi alla documentazione non sono destinati a rispondere a domande tecniche.

Eventuali idee da approfondire con la documentazione di AEM sono gradite come contributi. Tuttavia, commenti, problemi e richieste pull sono destinati solo ai *contributi* . Non sono destinati a rispondere alle tue domande su come usare AEM, implementare il tuo progetto AEM o risolvere problemi tecnici.

Eventuali domande sull’utilizzo di AEM o eventuali errori tecnici devono essere segnalate tramite il normale processo di assistenza tramite il portale [di assistenza](https://helpx.adobe.com/contact/enterprise-support.ec.html) Experience Cloud Enterprise o discusse nella community [di](https://forums.adobe.com/community/experience-cloud/marketing-cloud/experience-manager)Experience Manager.

***I contributi alla documentazione di AEM non sostituiscono l’Assistenza*** clienti Adobe e tutti questi contributi che richiedono risposte a domande relative al supporto verranno rifiutati.

### I contributi devono fare riferimento in modo chiaro alle pagine della documentazione interessate.

Se si crea un problema per suggerire miglioramenti alla documentazione, è necessario includere collegamenti alle pagine interessate. Se apri una segnalazione problema utilizzando il collegamento **Modifica pagina** di una pagina della documentazione, il problema verrà creato automaticamente con un collegamento alla pagina in questione.

Ciò non si applica alle richieste pull, in quanto le richieste pull per loro natura fanno riferimento alle pagine interessate.

## Linee guida sulla documentazione

Chiediamo che qualsiasi contributo alla nostra documentazione segua determinate linee guida di stile.

Seguendo queste linee guida è più semplice rivedere il proprio contributo e l&#39;integrazione nella documentazione è quindi più rapida.

### Lingua e stile

#### Lingua

* La documentazione di AEM è creata e gestita in inglese americano.
* Tenete delle frasi il più semplici possibile.
* Tenete la lingua chiara e concisa.

Ricorda che i lettori della documentazione di AEM sono in tutto il mondo e non possono essere madrelingua o parlanti inglesi fluenti. Evitare i colloquialismi e mantenere la lingua il più possibile chiara e semplice.

#### Segui il manuale di stile di Microsoft

[Il Manuale di Stile](https://docs.microsoft.com/en-us/style-guide/welcome/) di Microsoft è una guida libera allo stile della documentazione che si concentra sulla documentazione del software e la documentazione di AEM segue questa guida quando possibile.

### Formattazione

| Elemento | Stile |
|---|---|
| Elemento o opzione dell&#39;interfaccia utente | **grassetto** |
| Nome file, percorso, input utente, valori-parametro | `monospaced` |
| Codice, riga di comando | ```Code Block``` |

### Schermate

Le schermate devono essere utilizzate in modo appropriato e solo quando una descrizione testuale è insufficiente.

Non devono essere utilizzati marcatori o altre annotazioni nelle schermate (come cornici rosse, frecce o testo). In questo modo le schermate sono più facili da riutilizzare o replicare nelle versioni localizzate della documentazione.

### Riferimenti specifici per le versioni

Cercate di evitare, quando possibile, riferimenti diretti a una versione specifica nel contenuto della documentazione. Questo rende la documentazione più flessibile ed estensibile per le versioni future.

### Utilizzo di Day, AEM, CQ, CRX

Il prodotto deve sempre essere indicato dal nome completo di **Adobe Experience Manager** per la prima volta in un articolo e può essere successivamente denominato **AEM**.

Day, Day Software, CQ e CRX non devono essere utilizzati a meno che non sia inevitabile, ad esempio nei nomi delle classi o che faccia riferimento alla cronologia di AEM.