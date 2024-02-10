# ws-Protokoll

Derzeit ist die **ws-Protokollversion von Workerman 13**.

Workerman kann als Client dienen, der über das ws-Protokoll eine WebSocket-Verbindung herstellt, um mit einem entfernten WebSocket-Server in beiden Richtungen zu kommunizieren.

> **Anmerkung**
> Das ws-Protokoll kann nur als Client über AsyncTcpConnection verwendet werden und kann nicht als Protokoll für das WebSocket-Server-Listening verwendet werden. Das bedeutet, dass die folgende Schreibweise falsch ist.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Wenn Sie Workerman als WebSocket-Server verwenden möchten, verwenden Sie bitte das [WebSocket-Protokoll](about-websocket.md).

**Beispiel für das ws-Protokoll als WebSocket-Client-Protokoll:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Beim Start des Prozesses
$worker->onWorkerStart = function()
{
    // Verbindung zu einem entfernten WebSocket-Server über das WebSocket-Protokoll
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Senden eines Heartbeats mit einem Opcode von 0x9 an den Server alle 55 Sekunden (optional)
    $ws_connection->websocketPingInterval = 55;
    // Setzen der HTTP-Header (optional)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Setzen des Datentyps (optional)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB für Text BINARY_TYPE_ARRAYBUFFER für Binärdaten
    // Nachdem TCP-Handshake abgeschlossen ist (optional)
    $ws_connection->onConnect = function($connection){
        echo "TCP-Verbindung hergestellt\n";
    };
    // Nachdem der WebSocket-Handshake abgeschlossen ist (optional)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('Hallo');
    };
    // Wenn eine Nachricht vom entfernten WebSocket-Server empfangen wird
    $ws_connection->onMessage = function($connection, $data){
        echo "Empfangen: $data\n";
    };
    // Bei einem Verbindungsfehler, meist Fehler beim Verbinden mit dem entfernten WebSocket-Server (optional)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "Fehler: $msg\n";
    };
    // Wenn die Verbindung zum entfernten WebSocket-Server getrennt wird (optional, es wird empfohlen, Reconnect einzubeziehen)
    $ws_connection->onClose = function($connection){
        echo "Verbindung geschlossen und versuche erneut zu verbinden\n";
        // Wenn die Verbindung getrennt ist, 1 Sekunde später erneut verbinden
        $connection->reConnect(1);
    };
    // Nachdem alle oben genannten Rückrufe eingestellt wurden, die Verbindung ausführen
    $ws_connection->connect();
};
Worker::runAll();
```

Weitere Informationen finden Sie unter [Als ws-/wss-Client](../faq/as-wss-client.md).
