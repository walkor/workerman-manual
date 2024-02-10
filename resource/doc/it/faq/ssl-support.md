# Crittografia del trasporto - SSL/TLS

**Domanda:**

Come garantire la sicurezza della comunicazione con Workerman?

**Risposta:**

Un metodo conveniente è quello di aggiungere uno strato di crittografia [SSL](https://it.wikipedia.org/wiki/Transport_Layer_Security) sopra il protocollo di comunicazione. Ad esempio, i protocolli wss e [https](https://it.wikipedia.org/wiki/Https) si basano su trasmissioni crittografate con [SSL](https://it.wikipedia.org/wiki/Transport_Layer_Security) e sono molto sicuri. Workerman supporta nativamente [SSL](https://it.wikipedia.org/wiki/Transport_Layer_Security) (`richiede Workerman>=3.3.7`) e può essere attivato semplicemente impostando le opportune proprietà.

Naturalmente, gli sviluppatori possono anche implementare un proprio meccanismo di crittografia basato su determinati algoritmi di cifratura e decifratura.

## Metodo per attivare l'SSL in Workerman:

**Preparazione:**

1. Versione di Workerman non inferiore a 3.3.7
2. Estensione openssl installata in PHP
3. Certificato già richiesto (file pem/crt e file chiave) salvato in `/etc/nginx/conf.d/ssl`

**Codice:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// È consigliabile utilizzare un certificato richiesto
$context = array(
    'ssl' => array(
        'local_cert'        => '/etc/nginx/conf.d/ssl/server.pem', // può anche essere un file crt
        'local_pk'          => '/etc/nginx/conf.d/ssl/server.key',
        'verify_peer'       => false,
        'allow_self_signed' => true, // se si tratta di un certificato autogenerato, è necessario abilitare questa opzione
    )
);
// In questo caso, viene impostato il protocollo websocket, ma è possibile utilizzare anche protocolli HTTP o altri
$worker = new Worker('websocket://0.0.0.0:443', $context);
// Imposta il trasporto abilitando l'SSL
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```

## Abilitare l'indicazione del nome del server [SNI (Server Name Indication)](https://it.wikipedia.org/wiki/Server_Name_Indication) in Workerman
Permette di associare più certificati allo stesso IP e porta.

**Combinare file.pem e file .key del certificato:**
Unire i contenuti dei file .pem e dei rispettivi file .key, aggiungendo alla fine del file .pem il contenuto del file .key (oppure ignorandolo se la chiave privata è già inclusa nel file .pem).

*Si prega di notare che si tratta di un singolo certificato, non di copiare tutti i certificati in un unico file.*

Ad esempio, dopo la fusione, il contenuto del file pem per *host1.com.pem* potrebbe essere approssimativamente il seguente:
```text
-----BEGIN CERTIFICATE-----
MIIGXTCBA...
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIFBzCCA...
-----END CERTIFICATE-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAA....
-----END RSA PRIVATE KEY-----
```

**Codice:**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$context = array(
    'ssl' => array(
        'SNI_enabled' => true, // Abilita SNI
        'SNI_server_certs' => [ // Imposta più certificati
            'host1.com' => '/path/host1.com.pem', // Certificato 1
            'host2.com' => '/path/host2.com.pem', // Certificato 2
        ],
        'local_cert' => '/path/default.com.pem', // Certificato predefinito
        'local_pk'   => '/path/default.com.key',
    )
);
$worker = new Worker('websocket://0.0.0.0:443', $context);
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
