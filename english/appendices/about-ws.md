# The ws protocol

At present, the **ws protocol version of Workerman is 13**.

Workerman can act as a client to initiate a websocket connection through the ws protocol, connecting to a remote websocket server to achieve two-way communication.

> **Note**
> The ws protocol can only be used as a client through AsyncTcpConnection and cannot be used as a websocket server listening protocol. This means that the following code is incorrect.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

If you want to use Workerman as a websocket server, please use the [websocket protocol](about-websocket.md).

**Example of ws as a websocket client protocol:**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// When the process starts
$worker->onWorkerStart = function()
{
    // Connect to a remote websocket server using the websocket protocol
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Send a websocket heartbeat with opcode 0x9 to the server every 55 seconds (optional)
    $ws_connection->websocketPingInterval = 55;
    // Set HTTP headers (optional)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Set the data type (optional)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB for text, BINARY_TYPE_ARRAYBUFFER for binary
    // After TCP three-way handshake is complete (optional)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // After websocket handshake is complete (optional)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // When a message is received from the remote websocket server
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // When an error occurs in the connection, usually a failure to connect to the remote websocket server (optional)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // When the connection to the remote websocket server is closed (optional, it is recommended to include reconnection)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // If the connection is closed, reconnect after 1 second
        $connection->reConnect(1);
    };
    // After setting up the various callbacks above, initiate the connection operation
    $ws_connection->connect();
};
Worker::runAll();
```

For more information, refer to [Using ws/wss as the client](../faq/as-wss-client.md).
