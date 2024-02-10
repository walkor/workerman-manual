# Protocollo ws

Attualmente la versione del **protocollo ws di Workerman è la 13**.

Workerman può essere utilizzato come client per stabilire una connessione websocket tramite il protocollo ws e collegarsi a un server websocket remoto per realizzare una comunicazione bidirezionale.

> **Nota** Il protocollo ws può essere utilizzato solo come client tramite AsyncTcpConnection e non può essere usato come protocollo di ascolto del server websocket. In altre parole, il seguente codice è errato.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Se si desidera utilizzare Workerman come server websocket, è necessario utilizzare il [protocollo websocket](about-websocket.md).

**Esempio di protocollo ws come client websocket:**

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
    // Connessione al server websocket remoto tramite il protocollo websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Ogni 55 secondi invia un heartbeat websocket con opcode 0x9 al server (opzionale)
    $ws_connection->websocketPingInterval = 55;
    // Imposta gli header HTTP (opzionale)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Imposta il tipo di dati (opzionale)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB per testo, BINARY_TYPE_ARRAYBUFFER per binario
    // Quando il TCP ha completato il three-way handshake (opzionale)
    $ws_connection->onConnect = function($connection){
        echo "tcp connesso\n";
    };
    // Quando il websocket ha completato il handshake (opzionale)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('ciao');
    };
    // Quando il server websocket remoto invia un messaggio
    $ws_connection->onMessage = function($connection, $data){
        echo "ricevuto: $data\n";
    };
    // Se si verifica un errore durante la connessione, di solito è un errore di connessione al server websocket remoto (opzionale)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "errore: $msg\n";
    };
    // Quando la connessione al server websocket remoto viene interrotta (opzionale, si consiglia di aggiungere un tentativo di riconnessione)
    $ws_connection->onClose = function($connection){
        echo "connessione chiusa e prova a riconnettersi\n";
        // Se la connessione viene interrotta, riconnettersi dopo 1 secondo
        $connection->reConnect(1);
    };
    // Dopo aver configurato tutti i callback sopra, eseguire l'operazione di connessione
    $ws_connection->connect();
};
Worker::runAll();
```

Per ulteriori informazioni, fare riferimento a [Come client ws/wss](../faq/as-wss-client.md).
