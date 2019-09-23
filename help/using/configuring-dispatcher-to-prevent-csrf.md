---
title: Configurazione del dispatcher per impedire attacchi CSRF
seo-title: Configurazione di Adobe AEM Dispatcher per impedire attacchi CSRF
description: Scoprite come configurare AEM Dispatcher per prevenire gli attacchi di contraffazione delle richieste tra siti.
seo-description: Scoprite come configurare Adobe AEM Dispatcher per prevenire attacchi di tipo "cross-site Request Forgery".
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: spedizioniere
content-type: riferimento
discoiquuid: d61d021e-b338-4a1d-91ee-5542755e931
translation-type: tm+mt
source-git-commit: 69edbe7608b46c93d238515e4223606eadad0ac4

---


# Configurazione del dispatcher per impedire attacchi CSRF{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM fornisce un framework volto a prevenire attacchi di tipo "cross-site Request Forgery". Per utilizzare correttamente questo framework, è necessario apportare le seguenti modifiche alla configurazione del dispatcher:

>[!NOTE]
>
>Accertatevi di aggiornare i numeri delle regole negli esempi seguenti in base alla configurazione esistente. Tenere presente che i dispatcher utilizzeranno l'ultima regola di corrispondenza per concedere o negare un'autorizzazione, quindi posizionare le regole vicino alla parte inferiore dell'elenco esistente.

1. Nella `/clientheaders` sezione author-farm.any e publish-farm.any, aggiungete la seguente voce in fondo all’elenco:\
   `CSRF-Token`
1. Nella sezione /filter del file `author-farm.any` e `publish-farm.any` o `publish-filters.any` , aggiungete la seguente riga per consentire le richieste `/libs/granite/csrf/token.json` tramite dispatcher.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. Nella `/cache /rules` sezione della `publish-farm.any`, aggiungere una regola per impedire al dispatcher di memorizzare il `token.json` file nella cache. Generalmente gli autori non passano nella cache, pertanto non è necessario aggiungere la regola al `author-farm.any`proprio interno.\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Per verificare che la configurazione funzioni, controlla il file dispatcher.log in modalità DEBUG per verificare che il file token.json non sia memorizzato nella cache e non sia bloccato dai filtri. Dovresti visualizzare messaggi simili a:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

Potete inoltre verificare che le richieste abbiano esito positivo nell'apache `access_log`. Le richieste per "/libs/granite/csrf/token.json devono restituire un codice di stato HTTP 200.
