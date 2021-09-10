---
title: Elenco di controllo della sicurezza di Dispatcher
seo-title: Elenco di controllo della sicurezza di Dispatcher
description: Elenco di controllo della sicurezza da completare prima di procedere alla produzione.
seo-description: Elenco di controllo della sicurezza da completare prima di procedere alla produzione.
uuid: 7bfa3202-03f6-48e9-8d2e-2a40e137ecbe
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: fbfafa55-c029-4ed7-ab3e-1bebfde18248
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
exl-id: 49009810-b5bf-41fd-b544-19dd0c06b013
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '653'
ht-degree: 100%

---

# Elenco di controllo della sicurezza di Dispatcher {#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe consiglia vivamente di completare il seguente elenco di controllo prima di procedere alla produzione.

>[!CAUTION]
>
>Devi anche completare l’elenco di controllo della sicurezza della tua versione di AEM prima di andare “live”. Fare riferimento alla corrispondente [documentazione di Adobe Experience Manager](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=it).

## Utilizza la versione più recente di Dispatcher {#use-the-latest-version-of-dispatcher}

Installa la versione più recente disponibile per la piattaforma. Aggiorna l’istanza di Dispatcher per utilizzare la versione più recente e usufruire dei miglioramenti apportati al prodotto e alla sicurezza. Vedi [Installazione di Dispatcher](dispatcher-install.md).

>[!NOTE]
>
>Nel file di registro di Dispatcher, puoi verificare l’attuale versione di Dispatcher installata.
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>Per trovare il file di registro, controlla la configurazione di Dispatcher in `httpd.conf`.

## Limita il numero dei client che possono eseguire il flushing della cache {#restrict-clients-that-can-flush-your-cache}

Adobe consiglia di [limitare il numero dei client che possono eseguire il flushing della cache.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## Abilita HTTPS per la sicurezza del livello di trasporto {#enable-https-for-transport-layer-security}

Adobe consiglia di abilitare il livello di trasporto HTTPS sia sulle istanze Autore che Publish.

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:41:28.841-0400

<p>Recommended to have SSL termination, front end SSL.</p> 
<p>Question is do we want to have SSL communication between dispatcher and AEM instances (publish and/or author).</p> 
<p>We might want to have two items:</p> 
<ul> 
 <li>MUST HTTPS clients -&gt; dispatcher / load balancer</li> 
 <li>NICE load balancer -&gt; dispatcher<br /> </li> 
 <li>NICE dispatcher -&gt; instances if sensitive information such as credit cards / or infrastructure requirements such as DMZ</li> 
</ul>

 -->

## Limita l’accesso {#restrict-access}

Quando configuri Dispatcher, devi limitare il più possibile l’accesso esterno. Vedi [Esempio di sezione /filter](dispatcher-configuration.md#main-pars_184_1_title) nella documentazione di Dispatcher.

## Accertati che l’accesso agli URL amministrativi sia interdetto {#make-sure-access-to-administrative-urls-is-denied}

Utilizza i filtri per bloccare l’accesso esterno a qualsiasi URL amministrativo, ad esempio alla console Web.

Per un elenco degli URL da bloccare, vedi [Verifica della sicurezza di Dispatcher](dispatcher-configuration.md#testing-dispatcher-security).

## Utilizza gli elenchi Consentiti invece degli elenchi Bloccati {#use-allowlists-instead-of-blocklists}

Gli elenchi Consentiti permettono un migliore controllo degli accessi, in quanto presuppongono che tutte le richieste di accesso debbano essere negate, a meno che non facciano parte esplicitamente dell’elenco Consentiti. Questo modello offre un controllo più restrittivo sulle nuove richieste che potrebbero non essere state ancora esaminate o prese in considerazione durante una determinata fase della configurazione.

## Esegui Dispatcher con un utente di sistema dedicato {#run-dispatcher-with-a-dedicated-system-user}

Quando configuri il Dispatcher, accertati che il server web sia eseguito da un utente dedicato con privilegi minimi. Si consiglia di concedere l’accesso in scrittura solo alla cartella della cache di Dispatcher.

Inoltre, gli utenti IIS devono configurare il proprio sito web come segue:

1. Nell’impostazione del percorso fisico per il sito web, seleziona **Connetti come utente specifico**.
1. Imposta l’utente.

## Previeni gli attacchi Denial of Service (DoS) {#prevent-denial-of-service-dos-attacks}

Un attacco Denial of Service (DoS) è un tentativo di rendere la risorsa di un computer indisponibile per gli utenti a cui è destinata.

A livello di Dispatcher, esistono due metodi di configurazione per evitare gli attacchi DoS: [](https://experienceleague.adobe.com/docs/?lang=it#/filter (Filtri))

* Utilizza il modulo mod_rewrite (ad esempio, [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) per eseguire le convalide degli URL (se le regole del modello URL non sono troppo complesse).

* Impedisci a Dispatcher di memorizzare in cache gli URL con estensioni fittizie utilizzando dei [filtri](dispatcher-configuration.md#configuring-access-to-conten-tfilter).\
   Ad esempio, modifica le regole di caching per limitare il caching ai tipi mime previsti, ad esempio:

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

   È possibile visualizzare un esempio di file di configurazione per [limitare l’accesso esterno](#restrict-access), che include limitazioni per i tipi mime.

Per abilitare in modo sicuro la funzionalità completa sulle istanze Publish, configura i filtri per impedire l’accesso ai seguenti nodi:

* `/etc/`
* `/libs/`

Quindi, configura i filtri per consentire l’accesso ai seguenti percorsi dei nodi:

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS, CSS e JSON)
* `/libs/cq/security/userinfo.json` (Informazioni utente CQ)
* `/libs/granite/security/currentuser.json` (**i dati non devono essere memorizzati in cache**)

* `/libs/cq/i18n/*` (Internalizzazione)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## Configura Dispatcher per impedire gli attacchi CSRF {#configure-dispatcher-to-prevent-csrf-attacks}

AEM fornisce un [framework](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=it) per prevenire gli attacchi di tipo Cross-Site Request Forgery. Per utilizzare correttamente questo framework, devi inserire nell’elenco Consentiti il supporto per i token CSRF in Dispatcher. Per farlo, segui questi passaggi:

1. Crea un filtro per consentire il percorso `/libs/granite/csrf/token.json`;
1. Aggiungi l’intestazione `CSRF-Token` alla sezione `clientheaders` della configurazione di Dispatcher.

## Previeni il clickjacking {#prevent-clickjacking}

Per prevenire il clickjacking, ti consigliamo di configurare il server web per fornire l’intestazione HTTP `X-FRAME-OPTIONS` impostata su `SAMEORIGIN`.

Per ulteriori [informazioni sul clickjacking, vedi il sito OWASP](https://www.owasp.org/index.php/Clickjacking).

## Esegui un test di penetrazione {#perform-a-penetration-test}

Adobe consiglia vivamente di eseguire un test di penetrazione dell’infrastruttura AEM prima di procedere alla produzione.
