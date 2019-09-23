---
title: Elenco Di Controllo Del Dispatcher
seo-title: Elenco Di Controllo Del Dispatcher
description: Elenco di controllo di sicurezza da completare prima di iniziare la produzione.
seo-description: Elenco di controllo di sicurezza da completare prima di iniziare la produzione.
uuid: 7bfa3202-03f6-48e9-8d2e-2a40e137ecbe
contentOwner: Utente
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: spedizioniere
content-type: riferimento
discoiquuid: fbfafa55-c029-4ed7-ab3e-1bebfde18248
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 6d3ff696780ce55c077a1d14d01efeaebcb8db28

---


# Elenco Di Controllo Del Dispatcher{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Il dispatcher come sistema front-end offre un ulteriore livello di sicurezza all’infrastruttura Adobe Experience Manager. Adobe consiglia vivamente di completare il seguente elenco di controllo prima di iniziare la produzione.

>[!CAUTION]
>
>Devi anche completare l’elenco di controllo di sicurezza della tua versione di AEM prima di iniziare a lavorare. Consulta la documentazione [di](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html)Adobe Experience Manager corrispondente.

## Usa la versione più recente del dispatcher {#use-the-latest-version-of-dispatcher}

Installate la versione più recente disponibile disponibile per la piattaforma in uso. Devi aggiornare l'istanza del dispatcher per utilizzare la versione più recente per sfruttare i miglioramenti apportati a livello di prodotti e sicurezza. Consultate [Installazione del dispatcher](dispatcher-install.md).

>[!NOTE]
>
>È possibile controllare la versione corrente dell'installazione del dispatcher esaminando il file di registro del dispatcher.
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>Per trovare il file di registro, ispezionare la configurazione del dispatcher nella `httpd.conf`.

## Limita client in grado di cancellare la cache {#restrict-clients-that-can-flush-your-cache}

Adobe consiglia di [limitare i client in grado di cancellare la cache.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## Abilita HTTPS per la sicurezza del livello di trasporto {#enable-https-for-transport-layer-security}

Adobe consiglia di abilitare il livello di trasporto HTTPS sia per le istanze di creazione che per quelle di pubblicazione.

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

## Limita accesso {#restrict-access}

Durante la configurazione del dispatcher è necessario limitare il più possibile l'accesso esterno. Consulta [Esempio /filter Section](dispatcher-configuration.md#main-pars_184_1_title) nella documentazione del dispatcher.

## Assicurati che l’accesso agli URL amministrativi sia negato {#make-sure-access-to-administrative-urls-is-denied}

Accertatevi di utilizzare i filtri per bloccare l'accesso esterno a qualsiasi URL amministrativo, come la console Web.

Consultate [Verifica della sicurezza](dispatcher-configuration.md#testing-dispatcher-security) del dispatcher per un elenco di URL da bloccare.

## Usa whitelist invece delle blacklist {#use-whitelists-instead-of-blacklists}

Le whitelist rappresentano un modo migliore per fornire il controllo dell'accesso in quanto, di per sé, presuppongono che tutte le richieste di accesso debbano essere negate a meno che non facciano parte esplicitamente della whitelist. Questo modello offre un controllo più restrittivo sulle nuove richieste che potrebbero non essere state ancora esaminate o prese in considerazione durante una determinata fase di configurazione.

## Eseguire il dispatcher con un utente di sistema dedicato {#run-dispatcher-with-a-dedicated-system-user}

Durante la configurazione del dispatcher, accertatevi che il server Web sia eseguito da un utente dedicato con meno privilegi. Si consiglia di concedere l’accesso in scrittura solo alla cartella della cache del dispatcher.

Inoltre, gli utenti IIS devono configurare il proprio sito Web come segue:

1. Nell’impostazione del percorso fisico per il sito Web, selezionate **Connect come utente** specifico.
1. Impostate l’utente.

## Impedire attacchi Denial of Service (DoS) {#prevent-denial-of-service-dos-attacks}

Un attacco Denial of Service (DoS) è un tentativo di rendere una risorsa computer non disponibile agli utenti previsti.

A livello di dispatcher, sono disponibili due metodi di configurazione per prevenire attacchi DoS: [](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (Filtri))

* Utilizzate il modulo mod_rewrite (ad esempio, [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) per eseguire convalide URL (se le regole del pattern URL non sono troppo complesse).

* Impedisci al dispatcher di memorizzare nella cache gli URL con estensioni spurie utilizzando [i filtri](dispatcher-configuration.md#configuring-access-to-conten-tfilter).\
   Ad esempio, modificate le regole di caching per limitare il caching ai tipi mime previsti, ad esempio:

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`
   Un file di configurazione di esempio può essere visualizzato per [limitare l'accesso](#restrict-access)esterno, che include restrizioni per i tipi mime.

Per abilitare tutte le funzionalità nelle istanze di pubblicazione, configurate i filtri in modo da impedire l’accesso ai seguenti nodi:

* `/etc/`
* `/libs/`

Quindi configurate i filtri per consentire l'accesso ai seguenti percorsi dei nodi:

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS, CSS e JSON)
* `/libs/cq/security/userinfo.json` (informazioni utente CQ)
* `/libs/granite/security/currentuser.json` (**i dati non devono essere memorizzati nella cache**)

* `/libs/cq/i18n/*` (Internalizzazione)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## Configurare il dispatcher per impedire attacchi CSRF {#configure-dispatcher-to-prevent-csrf-attacks}

AEM fornisce un [framework](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps) per prevenire attacchi di tipo "cross-site Request Forgery". Per utilizzare correttamente questo framework, è necessario inserire in una whitelist il supporto per i token CSRF nel dispatcher. È possibile eseguire questa operazione tramite:

1. Creazione di un filtro per consentire il `/libs/granite/csrf/token.json` percorso;
1. Aggiungete l’ `CSRF-Token` intestazione alla `clientheaders` sezione della configurazione del dispatcher.

## Impedisci il clickjacking {#prevent-clickjacking}

Per evitare il clickjacking, si consiglia di configurare il server Web in modo che fornisca l’intestazione `X-FRAME-OPTIONS` HTTP impostata su `SAMEORIGIN`.

Per ulteriori [informazioni sul clickjacking, vedere il sito](https://www.owasp.org/index.php/Clickjacking)OWASP.

## Eseguire un test di penetrazione {#perform-a-penetration-test}

Adobe consiglia vivamente di eseguire un test di penetrazione dell’infrastruttura AEM prima di iniziare la produzione.

