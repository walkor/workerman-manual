# Come client ws/wss

A volte è necessario che workerman agisca come client per connettersi a un server utilizzando il protocollo ws/wss e interagire con esso.
Di seguito è riportato un esempio.

## workerman come client ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // Dopo il successo della handshake del websocket
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // Quando viene ricevuto un messaggio
    $con->onMessage = function (AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## workerman come client wss (ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // SSL richiede l'accesso alla porta 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Imposta il metodo di accesso con crittografia ssl, rendendolo wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## workerman come client wss (ws+ssl) + certificato ssl locale

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Imposta l'indirizzo IP locale, la porta e il certificato SSL per accedere all'host remoto
    $context_option = array(
        // Opzioni SSL, vedere http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Percorso al certificato locale. Deve essere in formato PEM e contenere il certificato locale e la chiave privata.
            'local_cert'        => '/your/path/to/pemfile',
            // Password del file local_cert.
            'passphrase'        => 'your_pem_passphrase',
            // Consente o meno i certificati autofirmati.
            'allow_self_signed' => true,
            // Verifica del certificato SSL richiesta o meno.
            'verify_peer'       => false
        )
    );

    // SSL richiede l'accesso alla porta 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Imposta il metodo di accesso con crittografia ssl, rendendolo wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Altre impostazioni

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Quando il processo è avviato
$worker->onWorkerStart = function()
{
    // Connettersi a un server websocket remoto utilizzando il protocollo websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Inviare un heartbeat websocket con opcode 0x9 ogni 55 secondi al server
    $ws_connection->websocketPingInterval = 55;
    // Intestazione personalizzata
    $ws_connection->headers = ['token' => 'value'];
    // Imposta il tipo di dato. Per impostazione predefinita, BINARY_TYPE_BLOB è il testo
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB per testo, BINARY_TYPE_ARRAYBUFFER per binario
    // Dopo il completamento della handshake TCP
    $ws_connection->onConnect = function($connection){
        echo "TCP connesso\n";
    };
    // Dopo il completamento della handshake websocket
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Quando il server websocket remoto invia un messaggio
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // Gestione degli errori, di solito quando c'è un errore di connessione al server websocket remoto
    $ws_connection->onError = function($connection, $code, $msg){
        echo "errore: $msg\n";
    };
    // Quando la connessione al server websocket remoto viene chiusa
    $ws_connection->onClose = function($connection){
        echo "Connessione chiusa, tentativo di riconnessione\n";
        // Se la connessione viene chiusa, riconnettersi dopo 1 secondo
        $connection->reConnect(1);
    };
    // Dopo aver configurato tutti i callback sopra, eseguire l'operazione di connessione
    $ws_connection->connect();
};
Worker::runAll();
```
