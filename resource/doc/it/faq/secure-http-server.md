# Creare un servizio https

**Domanda:**

Come posso creare un servizio https con Workerman, in modo che i client possano comunicare tramite il protocollo https?

**Risposta:**

Il protocollo https è essenzialmente costituito da http + SSL, ovvero si sovrappone allo strato http aggiungendo SSL. Workerman supporta sia il protocollo http che SSL (necessario Workerman versione >=3.3.7), quindi è sufficiente abilitare SSL sopra il protocollo http per supportare il protocollo https.

Ci sono due modi comuni per abilitare https con Workerman, uno è abilitare direttamente SSL con Workerman, l'altro è utilizzare nginx come proxy SSL. Si può scegliere uno dei due metodi, non entrambi contemporaneamente.

## Abilitare SSL con Workerman

**Preparazione:**

1. Versione di Workerman >=3.3.7

2. PHP con estensione openssl installata

3. Certificato (file pem/crt e chiave) già ottenuti e posizionati in /etc/nginx/conf.d/ssl

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Si consiglia di ottenere un certificato valido
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // può anche essere un file crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, //è necessario abilitare questa opzione per i certificati self-signed
    )
);

// Qui viene impostato il protocollo http
$worker = new Worker('http://0.0.0.0:443', $context);
// Impostare il trasporto su ssl, trasformando http+SSL in https
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

Con il codice sopra, con Workerman è stato creato un servizio https, consentendo ai client di comunicare con Workerman tramite il protocollo https con crittografia sicura.

**Test:**

Digitare l'indirizzo nel browser come `https://domain:443` per testare la connessione.

**Nota:**

1. La porta https deve essere raggiunta tramite il protocollo https e non con http.

2. Di solito il certificato è associato a un dominio specifico, quindi durante il test bisogna utilizzare il dominio anziché l'indirizzo IP.

3. Se non si riesce a connettersi tramite https, controllare il firewall del server.

## Utilizzare nginx come proxy SSL

Oltre a utilizzare SSL con Workerman, si può anche utilizzare nginx come proxy SSL per abilitare https.

> **Attenzione**
> Il proxy SSL di nginx e l'impostazione di SSL con Workerman sono alternative, non possono essere attivate contemporaneamente.

Il principio e il flusso della comunicazione sono i seguenti:

1. Il client avvia la connessione https verso nginx.

2. Nginx converte i dati del protocollo https in http e li inoltra alla porta http di Workerman.

3. Workerman, ricevuti i dati, li elabora e restituisce i dati del protocollo http a nginx.

4. Nginx converte nuovamente i dati del protocollo http in https e li inoltra al client.

### Configurazione di Nginx

**Requisiti preliminari e preparazione:**

1. Si suppone che Workerman stia ascoltando sulla porta 8181 (protocollo http).

2. Il certificato (file pem/crt e chiave) è già stato ottenuto e posizionato in /etc/nginx/conf.d/ssl.

3. Si prevede di utilizzare nginx per aprire la porta 443 per offrire i servizi di proxy wss (la porta può essere modificata a seconda delle esigenze).

**La configurazione di nginx è simile a quanto segue:**

```nginx
upstream workerman {
    server 127.0.0.1:8181;
    keepalive 10240;
}

server {
  listen 443;
  server_name domain.com;
  access_log off;

  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl/server.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl/server.key;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols SSLv3 SSLv2 TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;

  location / {
    proxy_pass http://workerman;
    proxy_http_version 1.1;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header Connection "";
  }
}
```
