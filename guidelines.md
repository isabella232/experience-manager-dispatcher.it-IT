---
source-git-commit: 053f90d9a0574c7d470cec6794498919e3340713
workflow-type: ht
source-wordcount: '752'
ht-degree: 100%

---
# Linee guida per contribuire alla documentazione di Adobe Experience Manager

## Filosofia della documentazione

Sappiamo che gli utenti di Adobe Experience Manager lavorano in ambienti altamente competitivi per creare esperienze digitali che li distinguano dalla concorrenza. Pertanto, è fondamentale che, quando Adobe offre nuovi strumenti avanzati in AEM, questi siano integrati da una documentazione accurata e chiara che consenta al cliente di sfruttare immediatamente il proprio investimento in AEM e massimizzare il ROI.

Per la documentazione di AEM, l’obiettivo è renderla disponibile agli utenti della soluzione il prima possibile. Viene quindi data priorità a una documentazione accurata e fruibile, sottoposta ad aggiornamento e miglioramento continuo.

## Contributi alla documentazione

Al fine di migliorare continuamente la documentazione di AEM, apprezziamo il contributo dell’intera comunità di utenti AEM. I miglioramenti apportati alla documentazione, che derivino da richieste o segnalazioni di problemi, possono essere correzioni, chiarimenti, ampliamenti ed esempi aggiuntivi.

## Standard della documentazione

Pur accogliendo con favore i contributi alla documentazione di AEM, qualsiasi contributo, sotto forma di richiesta o segnalazione di un problema, deve essere conforme ai nostri standard in merito ai contributi e alla documentazione.

I contributi che non soddisfano tali standard possono essere rifiutati.

### Documentiamo i casi di utilizzo standard.

La documentazione di AEM riguarda i casi di utilizzo standard. I casi di utilizzo che esulano dall’ambito di installazione e utilizzo standard del prodotto non rientrano nella documentazione di AEM.

### In genere non documentiamo i bug o le loro soluzioni.

La documentazione di AEM riguarda i casi di utilizzo standard. Per questo motivo, i bug, gli effetti causati da bug e le soluzioni per ovviare ai bug non sono generalmente documentati.

Fanne eccezione le note sulla versione, in cui possono essere elencati i problemi noti con possibili soluzioni approvate dal team AEM Product Management.

### I contributi alla documentazione non devono essere usati per rispondere a domande tecniche.

È gradita come contributo qualsiasi idea che punti al miglioramento della documentazione di AEM. Tuttavia, commenti, problemi e richieste sono intesi solo come *contributi*. Non hanno l’obiettivo di rispondere a domande su come usare AEM, come implementare un progetto AEM o risolvere problemi tecnici.

Eventuali domande sull’utilizzo di AEM o su errori tecnici possono essere segnalate attraverso il normale processo di assistenza tramite il [portale di assistenza Enterprise Support di Experience Cloud](https://helpx.adobe.com/it/contact/enterprise-support.ec.html) o discusse nella [community di Experience Manager](https://forums.adobe.com/community/experience-cloud/marketing-cloud/experience-manager).

***I contributi alla documentazione di AEM non sostituiscono l’Assistenza clienti di Adobe***. Pertanto, verranno rifiutati tutti i contributi che richiedono risposte a domande correlate al supporto.

### I contributi devono fare riferimento in modo chiaro alle pagine della documentazione interessate.

Se apri una segnalazione per suggerire miglioramenti alla documentazione, devi includere i collegamenti alle pagine interessate. Se apri una segnalazione utilizzando il collegamento **Modifica pagina** di una pagina della documentazione, il problema verrà creato automaticamente con un collegamento alla pagina in questione.

Ciò non si applica alle richieste pull, che per loro natura fanno riferimento alle pagine interessate.

## Linee guida per la documentazione

Chiediamo che qualsiasi contributo alla nostra documentazione segua determinate linee guida di stile.

L’adesione a queste linee guida semplifica la revisione del contributo proposto e ne velocizza quindi l’integrazione nella documentazione.

### Lingua e stile

#### Lingua

* La documentazione di AEM è creata e mantenuta in inglese americano.
* Usa frasi quanto più semplici possibile.
* Usa un linguaggio chiaro e conciso.

Ricorda che la documentazione di AEM viene letta da lettori in tutto il mondo, che potrebbero non essere di madrelingua inglese o non parlare inglese correntemente. Evita i colloquialismi e usa un linguaggio chiaro e semplice.

#### Segui il manuale di stile di Microsoft

Il [Manuale di stile di Microsoft](https://docs.microsoft.com/it-it/style-guide/welcome/) è una guida di stile per la documentazione disponibile gratuitamente, specifica per la documentazione di software. La documentazione di AEM segue questa guida quando possibile.

### Formattazione

| Elemento | Stile |
|---|---|
| Elemento o opzione dell’interfaccia utente | **grassetto** |
| Nome file, percorso, input utente, valori di parametri | `monospaced` |
| Codice, riga di comando | ```Code Block``` |

### Schermate

Non fare un uso eccessivo di schermate e usale solo quando una descrizione testuale è insufficiente.

Nelle schermate non devono essere utilizzati marcatori o altre annotazioni (come riquadri rossi, frecce o testo). In questo modo le schermate possono essere riutilizzate o replicate più facilmente nelle versioni localizzate della documentazione.

### Riferimenti a specifiche versioni

È meglio evitare, quando possibile, riferimenti diretti a una versione specifica nel contenuto della documentazione. In tal modo la documentazione sarà più flessibile e potrà essere estesa anche a versioni future.

### Utilizzo di Day, AEM, CQ, CRX

La prima volta che viene citato in un articolo, il prodotto deve sempre essere indicato con il suo nome completo **Adobe Experience Manager**; successivamente può essere usato **AEM**.

Day, Day Software, CQ e CRX non devono essere utilizzati a meno che non sia inevitabile, ad esempio nei nomi di classi o in riferimenti specifici a versioni storiche di AEM.