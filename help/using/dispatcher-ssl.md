---
title: Utilizzo di SSL con dispatcher
seo-title: Utilizzo di SSL con dispatcher
description: Scopri come configurare il dispatcher per comunicare con AEM tramite le connessioni SSL.
seo-description: Scopri come configurare il dispatcher per comunicare con AEM tramite le connessioni SSL.
uuid: 1 a 8 f 448 c-d 3 d 8-4798-a 5 cb -9579171171 ed
contentOwner: Utente
products: SG_ EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: riferimento
discoiquuid: 771 cfd 85-6 c 26-4 ff 2-a 3 fe-dff 8 d 8 f 7920 b
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# Utilizzo di SSL con dispatcher {#using-ssl-with-dispatcher}

Utilizzate connessioni SSL tra dispatcher e il computer di rendering:

* [SSL unidirezionale](dispatcher-ssl.md#main-pars-title-1)
* [SSL reciproca](dispatcher-ssl.md#main-pars-title-2)

>[!NOTE]
>
>Le operazioni relative ai certificati SSL sono associate a prodotti di terze parti. Non sono coperti dal contratto Adobe Platinum Maintenance and Support.

## Usa SSL quando il dispatcher si connette ad AEM {#use-ssl-when-dispatcher-connects-to-aem}

Configurate il dispatcher per comunicare con l&#39;istanza di rendering AEM o CQ utilizzando le connessioni SSL.

Prima di configurare il dispatcher, configurate AEM o CQ per utilizzare SSL:

* AEM 6.2: [Abilitazione HTTP su SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1: [Abilitazione HTTP su SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* Versioni precedenti di AEM: vedere [questa pagina](https://helpx.adobe.com/experience-manager/aem-previous-versions.html).

### Intestazioni richiesta SSL {#ssl-related-request-headers}

Quando il dispatcher ricalcola una richiesta HTTPS, il dispatcher include le seguenti intestazioni nella richiesta successiva inviata ad AEM o CQ:

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

Una richiesta tramite Apache -2.2 con `mod_ssl` include intestazioni simili al seguente esempio:

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### Configurazione del dispatcher per l&#39;utilizzo di SSL {#configuring-dispatcher-to-use-ssl}

Per configurare il dispatcher in modo che si connetta a AEM o CQ su SSL, il [dispatcher. Qualsiasi](dispatcher-configuration.md) file richiede le seguenti proprietà:

* Host virtuale che gestisce le richieste HTTPS.
* La `renders` sezione dell&#39;host virtuale include un elemento che identifica il nome host e la porta dell&#39;istanza CQ o AEM che utilizza HTTPS.
* `renders` L&#39;elemento include una proprietà denominata `secure` value `1`.

Nota: Se necessario, create un altro host virtuale per gestire le richieste HTTP.

Il seguente dispatcher di esempio. Qualsiasi file mostra i valori delle proprietà per la connessione HTTPS a un&#39;istanza CQ in esecuzione sull&#39;host `localhost` e sulla porta `8443`:

```
/farms
{
   /secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTPS requestss
         "https://*"
      }
      /renders
      {
      /0001
         {
            # hostname or IP of the render
            /hostname "localhost"
            # port of the render
            /port "8443"
            # connect via HTTPS
            /secure "1"
         }
      }
     # the rest of the properties are omitted
   }

   /non-secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTP requests
         "https://*"
      }
      /renders
      {
         /0001
      {
         # hostname or IP of the render
         /hostname "localhost"
         # port of the render
         /port "4503"
      }
   }
    # the rest of the properties are omitted
}
```

## Configurazione di Mutl SSL tra dispatcher e AEM {#configuring-mutual-ssl-between-dispatcher-and-aem}

Configurate le connessioni tra Dispatcher e il computer di rendering (in genere un&#39;istanza di pubblicazione AEM o CQ) per utilizzare SSL reciproca:

* Il dispatcher si collega all&#39;istanza di rendering su SSL.
* L&#39;istanza di rendering verifica la validità del certificato di Dispatcher.
* Il dispatcher verifica che l&#39;CA del certificato dell&#39;istanza di rendering sia affidabile.
* (Facoltativo) Il dispatcher verifica che il certificato dell&#39;istanza di rendering corrisponda all&#39;indirizzo del server dell&#39;istanza di rendering.

Per configurare SSL mutualmente, è necessario disporre di certificati firmati da un&#39;autorità di certificazione (CA) affidabile. I certificati autofirmati non sono adeguati. È possibile agire su CA o utilizzare i servizi di un CA di terze parti per firmare i certificati. Per configurare SSL mutualmente, è necessario disporre dei seguenti elementi:

* Certificati firmati per l&#39;istanza di rendering e il dispatcher
* Il certificato CA (se si agisce come CA)
* Librerie openssl per la generazione di CA, certificati e richieste di certificato.

Effettuate i seguenti passaggi per configurare SSL reciproca:

1. [Installa](dispatcher-install.md) la versione più recente di Dispatcher per la piattaforma. Utilizzate un binario di dispatcher che supporti SSL (SSL è nel nome del file, ad esempio dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar).
1. [Create o ottenete un certificato con firma CA](dispatcher-ssl.md#main-pars-title-3) per Dispatcher e l&#39;istanza di rendering.
1. [Create un archivio di chiavi contenente il certificato di rendering](dispatcher-ssl.md#main-pars-title-6) e configurate il servizio HTTP del rendering per utilizzarlo.
1. [Configura il modulo del server Web di Dispatcher](dispatcher-ssl.md#main-pars-title-4) per SSL reciproca.

### Creazione o acquisizione di certificati firmati tramite CA {#creating-or-obtaining-ca-signed-certificates}

Create o ottenete i certificati firmati CA che autenticano l&#39;istanza di pubblicazione e il dispatcher.

#### Creazione di un {#creating-your-ca}

Se si agisce come CA, utilizzare [openssl](https://www.openssl.org/) per creare l&#39;autorità di certificazione che firma il server e i certificati client. Devono essere installati le librerie openssl. Se si utilizza un CA di terze parti, non eseguire questa procedura.

1. Aprite un terminale e modificate la directory corrente nella directory in cui viene posizionato il file CA. sh, ad esempio `/usr/local/ssl/misc`.
1. Per creare l&#39;CA, inserire il comando seguente e quindi fornire i valori quando si esegue la propensione:

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >Diverse proprietà nel file openssl. cnf controllano il comportamento dello script CA. sh. È necessario modificare questo file come necessario prima di creare l&#39;CA.

#### Creazione dei certificati {#creating-the-certificates}

Utilizzate openssl per creare le richieste di certificati da inviare all&#39;CA di terze parti o per effettuare l&#39;accesso con CA.

Quando create un certificato, openssl utilizza la proprietà Nome comune per identificare il segnaposto del certificato. Per il certificato dell&#39;istanza di rendering, utilizzate il nome host del computer dell&#39;istanza come Nome comune se state configurando il dispatcher per accettare il certificato solo se corrisponde al nome host dell&#39;istanza Pubblica. (Vedete la proprietà [dispatchercheckpeercn](dispatcher-ssl.md#main-pars-title-11) .)

1. Aprite un terminale e modificate la directory corrente nella directory contenente il file CH. sh delle librerie openssl.
1. Inserite il comando seguente e fornite i valori quando richiesto. Se necessario, usate il nome host dell&#39;istanza di pubblicazione come Nome comune. Il nome host è il nome resolvable DNS per l&#39;indirizzo IP del rendering:

   ```shell
   ./CA.sh -newreq
   ```

   Se utilizzate un CA di terze parti, inviate il file newreq. pem all&#39;CA per la firma. Se si agisce come CA, proseguite con il passaggio 3.

1. Digitate il comando seguente per firmare il certificato utilizzando il certificato di CA:

   ```shell
   ./CA.sh -sign
   ```

   Nella directory sono creati due file denominati newcert. pem e newkey. pem che contengono i file di gestione CA. Rispettivamente il certificato pubblico e la chiave privata per il computer di rendering.

1. Rinomina newcert. pem in rendercert. pem e rinomina newkey. pem in renderkey. pem.
1. Ripetete i passaggi 2 e 3 per creare un nuovo certificato e una nuova chiave pubblica per il modulo Dispatcher. Accertatevi di utilizzare un Nome comune specifico per l&#39;istanza di dispatcher.
1. Rinomina newcert. pem in dispcert. pem e rinomina newkey. pem in modo che scada dispkey. pem.

### Configurazione SSL in Rendering computer {#configuring-ssl-on-the-render-computer}

Configurate SSL sull&#39;istanza di rendering utilizzando i file rendercert. pem e renderkey. pem.

#### Conversione del certificato di rendering in formato JKS {#converting-the-render-certificate-to-jks-format}

Utilizzate le seguenti virgole per convertire il certificato di rendering, ovvero un file PEM, in un file PKCS # 12. Includete anche il certificato dell&#39;CA che ha firmato il certificato di rendering:

1. In una finestra del terminale, modificate la directory corrente nella posizione del certificato di rendering e della chiave privata.
1. Inserite le seguenti virgole per convertire il certificato di rendering, ovvero un file PEM, in un file PKCS # 12. Includete anche il certificato dell&#39;CA che ha firmato il certificato di rendering:

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. Digitate il comando seguente per convertire il file PKCS # 12 nel formato JAVA (Java keystore):

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java Keystore viene creato utilizzando un alias predefinito. Se necessario, modificate l&#39;alias:

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### Aggiunta del cert CA al Truststore di Rendering {#adding-the-ca-cert-to-the-render-s-truststore}

Se si agisce come CA, importate il certificato CA in un archivio chiavi. Quindi, configurare la JVM che esegue l&#39;istanza di rendering per rendere affidabile l&#39;archivio chiavi.

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. Usate un editor di testo per aprire il file cacert. pem e rimuovere tutto il testo che precede la linea del followign:

   `-----BEGIN CERTIFICATE-----`

1. Utilizzate il comando seguente per importare il certificato in un archivio chiavi:

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. Per configurare la JVM che esegue l&#39;istanza di rendering per rendere affidabile l&#39;archivio chiavi, utilizzare la seguente proprietà di sistema:

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   Ad esempio, se utilizzate lo script crx-quickstart/bin/quickstart per avviare l&#39;istanza di pubblicazione, potete modificare la proprietà CQ_ JVM_ OPTS:

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### Configurazione dell&#39;istanza di rendering {#configuring-the-render-instance}

Utilizzate il certificato di rendering con le istruzioni contenute nella *sezione Abilita SSL nella sezione Istanza* pubblicazione per configurare il servizio HTTP dell&#39;istanza di rendering in modo che utilizzi SSL:

* AEM 6.2: [Abilitazione HTTP su SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1: [Abilitazione HTTP su SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* Versioni precedenti di AEM: vedere [questa pagina.](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)

### Configurazione SSL per il modulo Dispatcher {#configuring-ssl-for-the-dispatcher-module}

Per configurare il dispatcher in modo che utilizzi SSL mutualmente, preparate il certificato Dispatcher e configurate il modulo del server Web.

### Creazione di un certificato di dispatcher unificato {#creating-a-unified-dispatcher-certificate}

Combina il certificato di dispatcher e la chiave privata non crittografata in un unico file PEM. Usate un editor di testo o il `cat` comando per creare un file simile al seguente esempio:

1. Aprite un terminale e modificate la directory corrente nella posizione del file dispkey. pem.
1. Per decifrare la chiave privata, inserire il comando seguente:

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. Usate un editor di testo o il `cat` comando per combinare la chiave privata non crittografata e il certificato in un unico file simile al seguente esempio:

   ```xml
   -----BEGIN RSA PRIVATE KEY-----
   MIICxjBABgkqhkiG9w0B...
   ...M2HWhDn5ywJsX
   -----END RSA PRIVATE KEY-----
   -----BEGIN CERTIFICATE-----
   MIIC3TCCAk...
   ...roZAs=
   -----END CERTIFICATE-----
   ```

### Specifica del certificato da utilizzare per il dispatcher {#specifying-the-certificate-to-use-for-dispatcher}

Aggiungi le seguenti proprietà alla [configurazione](dispatcher-install.md#main-pars-55-35-1022) del modulo Dispatcher (in `httpd.conf`):

* `DispatcherCertificateFile`: Percorso del file del certificato unificato Dispatcher contenente il certificato pubblico e la chiave privata non crittografata. Questo file viene usato quando SSL Server richiede il certificato client Dispatcher.
* `DispatcherCACertificateFile`: Il percorso del file di certificato CA, utilizzato se il server SSL presenta un CA che non è considerato affidabile da parte di un&#39;autorità principale.
* `DispatcherCheckPeerCN`: Se attivare ( `On`) o disattivare ( `Off`) il controllo del nome host per i certificati server remoti.

Il codice seguente è una configurazione di esempio:

```xml
<IfModule disp_apache2.c>
  DispatcherConfig conf/dispatcher.any
  DispatcherLog    logs/dispatcher.log
  DispatcherLogLevel 3
  DispatcherNoServerHeader 0
  DispatcherDeclineRoot 0
  DispatcherUseProcessedURL 0
  DispatcherPassError 0
  DispatcherCertificateFile disp_unified.pem
  DispatcherCACertificateFile cacert.pem
  DispatcherCheckPeerCN On
</IfModule>
```

