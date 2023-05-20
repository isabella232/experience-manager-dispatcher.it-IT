---
title: Configurazione di Dispatcher per impedire attacchi CSRF
seo-title: Configuring Adobe AEM Dispatcher to Prevent CSRF Attacks
description: Scopri come configurare AEM Dispatcher per impedire attacchi Cross-Site Request Forgery.
seo-description: Learn how to configure Adobe AEM Dispatcher to prevent Cross-Site Request Forgery attacks.
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '225'
ht-degree: 100%

---

# Configurazione di Dispatcher per impedire attacchi CSRF {#configuring-dispatcher-to-prevent-csrf-attacks}

AEM fornisce un framework per prevenire gli attacchi di tipo Cross-Site Request Forgery. Per utilizzare correttamente questo framework, devi apportare le seguenti modifiche alla configurazione di Dispatcher:

>[!NOTE]
>
>Non dimenticare di aggiornare i numeri delle regole negli esempi sotto riportati in base alla configurazione esistente. Ricorda che Dispatcher utilizzerà l’ultima regola corrispondente per concedere o negare un’autorizzazione, pertanto posiziona le regole in fondo all’elenco esistente.

1. Nella sezione `/clientheaders` di dei file author-farm.any e publish-farm.any, aggiungi la seguente voce in fondo all’elenco:\
   `CSRF-Token`
1. Nella sezione /filters dei file `author-farm.any` e `publish-farm.any` o `publish-filters.any`, aggiungi la seguente riga per consentire le richieste di `/libs/granite/csrf/token.json` tramite Dispatcher.\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. Nella sezione `/cache /rules` del file `publish-farm.any`, aggiungi una regola per impedire a Dispatcher di memorizzare in cache il file `token.json`. In genere, gli autori ignorano il caching, pertanto non è necessario aggiungere la regola nel file `author-farm.any`.\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

Per verificare che la configurazione funzioni, esamina il file dispatcher.log in modalità DEBUG per accertarti che il file token.json non sia memorizzato in cache e non sia bloccato da filtri. Dovresti vedere messaggi simili ai seguenti:\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

Puoi anche verificare che le richieste abbiano esito positivo nel file `access_log` del tuo modulo Apache. Le richieste di ``/libs/granite/csrf/token.json devono restituire il codice di stato HTTP 200.
