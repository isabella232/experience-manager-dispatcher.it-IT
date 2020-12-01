---
title: Configurazione di Dispatcher per impedire attacchi CSRF
seo-title: Configurazione  Adobe AEM Dispatcher per impedire attacchi CSRF
description: Scoprite come configurare AEM Dispatcher per prevenire attacchi di tipo Forgery su più siti.
seo-description: Scoprite come configurare  Adobe AEM Dispatcher per evitare attacchi di tipo Cross-Site Request Forgery.
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
translation-type: tm+mt
source-git-commit: 69edbe7608b46c93d238515e4223606eadad0ac4
workflow-type: tm+mt
source-wordcount: '246'
ht-degree: 4%

---


# Configurazione di Dispatcher per impedire attacchi CSRF{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM fornisce un framework per prevenire attacchi di contraffazione delle richieste cross-site. Per utilizzare correttamente questo framework, è necessario apportare le seguenti modifiche alla configurazione del dispatcher:

>[!NOTE]
>
>Accertatevi di aggiornare i numeri delle regole negli esempi seguenti in base alla configurazione esistente. Tenere presente che i dispatcher utilizzeranno l&#39;ultima regola di corrispondenza per concedere o negare un&#39;autorizzazione, quindi posizionare le regole vicino alla parte inferiore dell&#39;elenco esistente.

1. Nella sezione `/clientheaders` di author-farm.any e publish-farm.any, aggiungete la seguente voce in fondo all’elenco:\
   `CSRF-Token`
1. Nella sezione /filter del file `author-farm.any` e `publish-farm.any` o `publish-filters.any`, aggiungere la seguente riga per consentire le richieste di `/libs/granite/csrf/token.json` tramite il dispatcher.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. Nella sezione `/cache /rules` del `publish-farm.any`, aggiungere una regola per bloccare il dispatcher dal caching del file `token.json`. Generalmente gli autori non passano nella cache, pertanto non è necessario aggiungere la regola nel `author-farm.any`.\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Per verificare che la configurazione funzioni, controlla il file dispatcher.log in modalità DEBUG per verificare che il file token.json non sia memorizzato nella cache e non sia bloccato dai filtri. Dovresti visualizzare messaggi simili a:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

È inoltre possibile verificare che le richieste abbiano esito positivo nell&#39;apache `access_log`. Le richieste per &quot;/libs/granite/csrf/token.json devono restituire un codice di stato HTTP 200.
