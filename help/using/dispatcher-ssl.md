---
title: Utilizzo di SSL con Dispatcher
seo-title: Utilizzo di SSL con Dispatcher
description: Scoprite come configurare il dispatcher per comunicare con AEM utilizzando le connessioni SSL.
seo-description: Scoprite come configurare il dispatcher per comunicare con AEM utilizzando le connessioni SSL.
uuid: 1a8f448c-d3d8-4798-a5cb-9579171171ed
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 771cfd85-6c26-4ff2-a3fe-dff8d8f7920b
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f9fb0e94dbd1c67bf87463570e8b5eddaca11bf3
workflow-type: tm+mt
source-wordcount: '1375'
ht-degree: 0%

---


# Utilizzo di SSL con Dispatcher {#using-ssl-with-dispatcher}

Utilizzate le connessioni SSL tra il dispatcher e il computer di rendering:

* [SSL unidirezionale](#use-ssl-when-dispatcher-connects-to-aem)
* [SSL reciproco](#configuring-mutual-ssl-between-dispatcher-and-aem)

>[!NOTE]
>
>Le operazioni relative ai certificati SSL sono associate a prodotti di terze parti. Non sono coperti dal contratto Platinum Maintenance and Support del Adobe .

## Usa SSL quando il dispatcher si connette a AEM {#use-ssl-when-dispatcher-connects-to-aem}

Configurate il dispatcher per comunicare con l’istanza di rendering AEM o CQ mediante connessioni SSL.

Prima di configurare il dispatcher, configurate AEM o CQ per l’utilizzo di SSL:

* AEM 6.2: [Abilitazione di HTTP su SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1: [Abilitazione di HTTP su SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* Versioni precedenti AEM: vedi [questa pagina](https://helpx.adobe.com/experience-manager/aem-previous-versions.html).

### Intestazioni di richiesta correlate a SSL {#ssl-related-request-headers}

Quando il dispatcher riceve una richiesta HTTPS, il dispatcher include le seguenti intestazioni nella richiesta successiva che invia a AEM o CQ:

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

Una richiesta tramite Apache-2.4 con `mod_ssl` intestazioni simili al seguente esempio:

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### Configurazione del dispatcher per l&#39;utilizzo di SSL {#configuring-dispatcher-to-use-ssl}

Per configurare il dispatcher per la connessione con AEM o CQ tramite SSL, il file [dispatcher.any](dispatcher-configuration.md) richiede le seguenti proprietà:

* Un host virtuale che gestisce le richieste HTTPS.
* La `renders` sezione dell&#39;host virtuale include un elemento che identifica il nome host e la porta dell&#39;istanza CQ o AEM che utilizza HTTPS.
* L&#39; `renders` elemento include una proprietà denominata `secure` of value `1`.

Nota: Se necessario, create un altro host virtuale per la gestione delle richieste HTTP.

Il seguente file dispatcher.any di esempio mostra i valori delle proprietà per la connessione mediante HTTPS a un&#39;istanza CQ in esecuzione su host `localhost` e porta `8443`:

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

## Configurazione di SSL reciproco tra dispatcher e AEM {#configuring-mutual-ssl-between-dispatcher-and-aem}

Configurate le connessioni tra il Dispatcher e il computer di rendering (in genere un’istanza di pubblicazione AEM o CQ) per utilizzare SSL reciproco:

* Il dispatcher si connette all’istanza di rendering tramite SSL.
* L&#39;istanza di rendering verifica la validità del certificato del dispatcher.
* Dispatcher verifica che la CA del certificato dell&#39;istanza di rendering sia attendibile.
* (Facoltativo) Il dispatcher verifica che il certificato dell&#39;istanza di rendering corrisponda all&#39;indirizzo del server dell&#39;istanza di rendering.

Per configurare SSL reciproco, è necessario disporre di certificati firmati da un&#39;autorità di certificazione (CA) affidabile. I certificati autofirmati non sono adeguati. È possibile fungere da CA o utilizzare i servizi di una CA di terze parti per firmare i certificati. Per configurare SSL reciproco, sono necessari i seguenti elementi:

* Certificati firmati per l’istanza di rendering e il dispatcher
* Certificato CA (se si agisce come CA)
* Librerie OpenSSL per la generazione di CA, certificati e richieste di certificati.

Per configurare SSL reciproco, effettuate le seguenti operazioni:

1. [Installa](dispatcher-install.md) la versione più recente del dispatcher per la tua piattaforma. Utilizzate un file binario del dispatcher che supporta SSL (SSL si trova nel nome file, ad esempio dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar).
1. [Create o ottenete un certificato](dispatcher-ssl.md#main-pars-title-3) con firma CA per Dispatcher e l’istanza di rendering.
1. [Create un archivio di chiavi contenente il certificato](dispatcher-ssl.md#main-pars-title-6) di rendering e configurate il servizio HTTP del rendering per utilizzarlo.
1. [Configurare il modulo](dispatcher-ssl.md#main-pars-title-4) del server Web del dispatcher per SSL reciproco.

### Creazione o ottenimento di certificati con firma CA {#creating-or-obtaining-ca-signed-certificates}

Create o ottenete i certificati con firma CA che autenticano l’istanza di pubblicazione e il dispatcher.

#### Creazione di un CA {#creating-your-ca}

Se si agisce come CA, utilizzare [OpenSSL](https://www.openssl.org/) per creare l&#39;Autorità di certificazione che firma i certificati server e client. È necessario che siano installate le librerie OpenSSL. Se si utilizza una CA di terze parti, non eseguire questa procedura.

1. Aprite un terminale e modificate la directory corrente nella directory che contiene il file CA.sh, ad esempio `/usr/local/ssl/misc`.
1. Per creare la CA, immettete il comando seguente e fornite i valori quando richiesto:

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >Diverse proprietà nel file openssl.cnf controllano il comportamento dello script CA.sh. È necessario modificare il file come necessario prima di creare la CA.

#### Creazione dei certificati {#creating-the-certificates}

Utilizzate OpenSSL per creare le richieste di certificato da inviare alla CA di terze parti o per accedere con la CA.

Quando create un certificato, OpenSSL utilizza la proprietà Nome comune per identificare il titolare del certificato. Per il certificato dell&#39;istanza di rendering, usate il nome host del computer dell&#39;istanza come Nome comune se state configurando Dispatcher per accettare il certificato solo se corrisponde al nome host dell&#39;istanza Publish. (See the [DispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11) property.)

1. Aprite un terminale e passate alla directory che contiene il file CH.sh delle librerie OpenSSL.
1. Inserite il comando seguente e fornite i valori quando richiesto. Se necessario, usate il nome host dell’istanza pubblicata come Nome comune. Il nome host è il nome risolvibile DNS per l&#39;indirizzo IP del rendering:

   ```shell
   ./CA.sh -newreq
   ```

   Se utilizzi una CA di terze parti, invia il file newreq.pem alla CA per firmare. Se si agisce come CA, continuare a fare il punto 3.

1. Digitate il comando seguente per firmare il certificato utilizzando il certificato della CA:

   ```shell
   ./CA.sh -sign
   ```

   Nella directory che contiene i file di gestione CA vengono creati due file denominati newcert.pem e newkey.pem. Si tratta rispettivamente del certificato pubblico e della chiave privata per il computer di rendering.

1. Rinominate newcert.pem in rendercert.pem e rinominate newkey.pem in renderkey.pem.
1. Ripetere i passaggi 2 e 3 per creare un nuovo certificato e una nuova chiave pubblica per il modulo Dispatcher. Assicurarsi di utilizzare un Nome comune specifico per l&#39;istanza Dispatcher.
1. Rinominare newcert.pem in discert.pem e rinominare newkey.pem in display.pem.

### Configurazione di SSL nel computer di rendering {#configuring-ssl-on-the-render-computer}

Configurate SSL nell’istanza di rendering utilizzando i file rendercert.pem e renderkey.pem.

#### Conversione del certificato di rendering in formato JKS {#converting-the-render-certificate-to-jks-format}

Usate il seguente comando per convertire il certificato di rendering, che è un file PEM, in un file PKCS#12. Includete anche il certificato della CA che ha firmato il certificato di rendering:

1. In una finestra del terminale, modificate la directory corrente nella posizione del certificato di rendering e della chiave privata.
1. Digitate il seguente comando per convertire il certificato di rendering, che è un file PEM, in un file PKCS#12. Includete anche il certificato della CA che ha firmato il certificato di rendering:

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. Digitate il comando seguente per convertire il file PKCS#12 in formato Java KeyStore (JKS):

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java Keystore viene creato utilizzando un alias predefinito. Se necessario, modificate l’alias:

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### Aggiunta del certificato CA al TrustStore del rendering {#adding-the-ca-cert-to-the-render-s-truststore}

Se fungi da CA, importa il certificato CA in un archivio chiavi. Quindi, configurate la JVM che esegue l&#39;istanza di rendering in modo da rendere affidabile l&#39;archivio chiavi.

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. Usate un editor di testo per aprire il file cacert.pem e rimuovere tutto il testo che precede la riga seguente:

   `-----BEGIN CERTIFICATE-----`

1. Utilizzate il comando seguente per importare il certificato in un archivio di chiavi:

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. Per configurare la JVM che esegue l&#39;istanza di rendering in modo che sia affidabile per l&#39;archivio di chiavi, utilizzare la seguente proprietà di sistema:

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   Ad esempio, se utilizzate lo script crx-quickstart/bin/quickstart per avviare l&#39;istanza di pubblicazione, potete modificare la proprietà CQ_JVM_OPTS:

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### Configurazione dell’istanza di rendering {#configuring-the-render-instance}

Utilizzate il certificato di rendering con le istruzioni riportate nella sezione *Abilita SSL nella sezione Istanza* di pubblicazione per configurare il servizio HTTP dell’istanza di rendering in modo che utilizzi SSL:

* AEM 6.2: [Abilitazione di HTTP su SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1: [Abilitazione di HTTP su SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* Versioni precedenti AEM: vedi [questa pagina.](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)

### Configurazione di SSL per il modulo Dispatcher {#configuring-ssl-for-the-dispatcher-module}

Per configurare il dispatcher in modo che utilizzi SSL reciproco, prepara il certificato del dispatcher e configura il modulo del server Web.

### Creazione di un certificato del dispatcher unificato {#creating-a-unified-dispatcher-certificate}

Combinate il certificato dispatcher e la chiave privata non crittografata in un unico file PEM. Utilizzare un editor di testo o il `cat` comando per creare un file simile al seguente esempio:

1. Aprite un terminale e modificate la directory corrente nella posizione del file display.pem.
1. Per decrittografare la chiave privata, digitate il comando seguente:

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. Usate un editor di testo o il `cat` comando per combinare la chiave privata non crittografata con il certificato in un singolo file, simile al seguente esempio:

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

Aggiungi le seguenti proprietà alla configurazione [del modulo](dispatcher-install.md#main-pars-55-35-1022) Dispatcher (in `httpd.conf`):

* `DispatcherCertificateFile`: Percorso del file di certificato unificato del dispatcher, contenente il certificato pubblico e la chiave privata non crittografata. Questo file viene utilizzato quando il server SSL richiede il certificato del client del dispatcher.
* `DispatcherCACertificateFile`: Percorso del file del certificato CA, utilizzato se il server SSL presenta una CA non attendibile da un&#39;autorità radice.
* `DispatcherCheckPeerCN`: Se abilitare ( `On`) o disabilitare ( `Off`) il controllo del nome host per i certificati del server remoto.

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

