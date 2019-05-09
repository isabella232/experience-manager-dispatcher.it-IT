---
title: Elenco di controllo della sicurezza del dispatcher
seo-title: Elenco di controllo della sicurezza del dispatcher
description: Elenco di controllo della sicurezza da completare prima di iniziare la produzione.
seo-description: Elenco di controllo della sicurezza da completare prima di iniziare la produzione.
uuid: 7 bfa 3202-03 f 6-48 e 9-8 d 2 e -2 a 40 e 137 ecbe
contentOwner: Utente
products: SG_ EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: riferimento
discoiquuid: fbfafa 55-c 029-4 ed 7-ab 3 e -1 bebfde 18248
jcr-lastmodifiedby: remove-legacypath -6-1
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97

---


# Elenco di controllo della sicurezza del dispatcher{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Il dispatcher come sistema front-end offre un ulteriore livello di protezione all&#39;infrastruttura Adobe Experience Manager. Adobe consiglia vivamente di completare il seguente elenco di controllo prima di procedere alla produzione.

>[!CAUTION]
>
>Dovete anche completare l&#39;elenco di controllo protezione della versione di AEM prima di andare live. Consultate la documentazione corrispondente [di Adobe Experience Manager](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html).

## Utilizzare la versione più recente del dispatcher {#use-the-latest-version-of-dispatcher}

Installate la versione più recente disponibile per la piattaforma. Devi aggiornare l&#39;istanza di Dispatcher per utilizzare la versione più recente per usufruire dei miglioramenti di prodotto e sicurezza. Consultate [Installazione del dispatcher](dispatcher-install.md).

>[!NOTE]
>
>Puoi controllare la versione corrente dell&#39;installazione del dispatcher guardando il file di registro del dispatcher.
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>Per trovare il file di registro, ispezionate la configurazione di dispatcher in `httpd.conf`.

## Limita i client che possono svuotare la cache {#restrict-clients-that-can-flush-your-cache}

Adobe consiglia [di limitare i client che possono svuotare la cache.](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## Abilita HTTPS per protezione livello di trasporto {#enable-https-for-transport-layer-security}

Adobe consiglia di abilitare il livello di trasporto HTTPS sia sulle istanze di creazione che di pubblicazione.

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

Durante la configurazione del dispatcher, è necessario limitare il più possibile l&#39;accesso esterno. Consultate [l&#39;esempio /filter Sezione](dispatcher-configuration.md#main-pars_184_1_title) nella documentazione del dispatcher.

## Assicurati che l&#39;accesso agli URL amministrativi sia rifiutato {#make-sure-access-to-administrative-urls-is-denied}

Accertatevi di utilizzare i filtri per bloccare l&#39;accesso esterno ad eventuali URL amministrativi, ad esempio la console Web.

Consultate [Testing Dispatcher Security](dispatcher-configuration.md#testing-dispatcher-security) (Test di protezione del dispatcher) per un elenco di URL che devono essere bloccati.

## Utilizzare le whitelist invece di Blacklists {#use-whitelists-instead-of-blacklists}

Le whitelist rappresentano un modo migliore per fornire il controllo degli accessi in modo ereditato, presupponendo che tutte le richieste di accesso vengano rifiutate, a meno che non siano esplicitamente parte della whitelist. Questo modello fornisce un controllo più restrittivo sulle nuove richieste che potrebbero non essere state ancora controllate o prese in considerazione durante una determinata fase di configurazione.

## Eseguire dispatcher con un utente di sistema dedicato {#run-dispatcher-with-a-dedicated-system-user}

Quando configurate il dispatcher, assicuratevi che il server Web venga eseguito da un utente dedicato con meno privilegi. È consigliabile concedere solo l&#39;acronimo in scrittura alla cartella della cache del dispatcher.

Additionnaly, gli utenti IIS devono configurare il proprio sito Web come segue:

1. Nell&#39;impostazione percorso fisico per il sito Web, selezionate **Connect come utente specifico**.
1. Impostate l&#39;utente.

## Impedire attacchi di negazione di servizio (DoS) {#prevent-denial-of-service-dos-attacks}

Un attacco Denial of Service (DoS) è un tentativo di rendere una risorsa computer non disponibile agli utenti desiderati.

A livello di dispatcher, esistono due metodi di configurazione per evitare attacchi DoS: [](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (Filtri))

* Utilizzare il modulo mod_ riwrite (ad esempio [Apache 2.2](https://httpd.apache.org/docs/2.2/mod/mod_rewrite.html)) per eseguire le convalide URL (se le regole di pattern URL non sono troppo complesse).

* Impedite al dispatcher di memorizzare gli URL di cache con estensioni spuntate utilizzando [i filtri](dispatcher-configuration.md#configuring-access-to-conten-tfilter).\
   Ad esempio, modificate le regole di caching per limitare la memorizzazione nella cache ai tipi mime previsti, ad esempio:

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`
   Un esempio di file di configurazione può essere visualizzato [per limitare l&#39;accesso esterno](#restrict-access), incluse le limitazioni per i tipi mime.

Per abilitare in modo sicuro le funzionalità complete nelle istanze pubblicate, configurate i filtri per impedire l&#39;accesso ai seguenti nodi:

* `/etc/`
* `/libs/`

Quindi, configurate i filtri per consentire l&#39;accesso ai seguenti percorsi nodo:

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS, CSS e JSON)
* `/libs/cq/security/userinfo.json` (Informazioni utente CQ)
* `/libs/granite/security/currentuser.json`**(i dati non devono essere memorizzati nella cache**)

* `/libs/cq/i18n/*` (Internalization)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## Configurare il dispatcher per evitare attacchi CSRF {#configure-dispatcher-to-prevent-csrf-attacks}

AEM offre [un framework](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps) mirato a evitare attacchi Cross-Site Request Forgery. Per utilizzare correttamente questo framework, è necessario inserire nel dispatcher il supporto per token CSRF. Puoi eseguire questa operazione:

1. Creazione di un filtro per consentire il `/libs/granite/csrf/token.json` percorso;
1. Aggiungi l `CSRF-Token` &#39;intestazione alla `clientheaders` sezione della configurazione di Dispatcher.

## Impedisci clickjacking {#prevent-clickjacking}

Per evitare il clickjacking, consigliamo di configurare il webserver Web su cui è installato l&#39;intestazione `X-FRAME-OPTIONS` HTTP `SAMEORIGIN`.

Per ulteriori [informazioni sul clickjacking, consultate il sito OWASP](https://www.owasp.org/index.php/Clickjacking).

## Eseguire un test di penetrazione {#perform-a-penetration-test}

Adobe consiglia vivamente di eseguire un test di penetrazione dell&#39;infrastruttura AEM prima di procedere alla produzione.

