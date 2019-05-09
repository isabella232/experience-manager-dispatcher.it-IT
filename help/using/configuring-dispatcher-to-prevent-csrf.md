---
title: Configurazione del dispatcher per impedire attacchi CSRF
seo-title: Configurazione di Adobe AEM Dispatcher per impedire attacchi CSRF
description: Scopri come configurare il dispatcher AEM per evitare attacchi Cross-Site Request Forgery.
seo-description: Scopri come configurare Adobe AEM Dispatcher per evitare attacchi Cross-Site Request Forgery.
uuid: f 290 bdeb -54 e 2-4649-b 0 fc -6257 b 422 af 2 d
topic-tags: dispatcher
content-type: riferimento
discoiquuid: d 61 d 021 e-b 338-4 a 1 d -91 ee -55427557 e 931
translation-type: tm+mt
source-git-commit: 69edbe7608b46c93d238515e4223606eadad0ac4

---


# Configurazione del dispatcher per impedire attacchi CSRF{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM offre un framework mirato a evitare attacchi Cross-Site Request Forgery. Per utilizzare correttamente questo framework, dovete apportare le seguenti modifiche alla configurazione di dispatcher:

>[!NOTE]
>
>Assicuratevi di aggiornare i numeri di regole negli esempi seguenti in base alla configurazione esistente. Tenere presente che i dispatcher utilizzeranno l&#39;ultima regola corrispondente per concedere un&#39;autorizzazione o un rifiuto, quindi inserite le regole in fondo all&#39;elenco esistente.

1. Nella `/clientheaders` sezione di author-farm. any e publish-farm. any, aggiungete la voce seguente in fondo all&#39;elenco:\
   `CSRF-Token`
1. Nella sezione /filters di `author-farm.any` e `publish-farm.any` o `publish-filters.any` file, aggiungete la seguente riga per consentire le richieste `/libs/granite/csrf/token.json` tramite dispatcher.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. Sotto la `/cache /rules` sezione, `publish-farm.any`aggiungete una regola per bloccare il dispatcher dal caching `token.json` del file. In genere, gli autori superano il caching, pertanto non dovrebbe essere necessario aggiungere la `author-farm.any`regola.\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Per verificare che la configurazione funzioni, guarda il file dispatcher. log in modalità DEBUG per verificare che il file token. json non sia in fase di memorizzazione nella cache e non venga bloccato dai filtri. Dovresti visualizzare messaggi simili a:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

Potete inoltre convalidare che le richieste abbiano successo nell&#39;apache `access_log`. Le richieste per «/libs/granite/csrf/token. json devono restituire un codice di stato HTTP 200.
