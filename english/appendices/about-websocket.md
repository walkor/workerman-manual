# WebSocket Protocol

Currently, the **WebSocket protocol version of Workerman is 13**.

WebSocket protocol is a new protocol in HTML5. It implements full-duplex communication between the browser and the server.

## Relationship between WebSocket and TCP

WebSocket, like HTTP, is an application-layer protocol based on TCP transmission. WebSocket itself is not directly related to traditional sockets and cannot be equated with them.

## WebSocket Protocol Handshake

The WebSocket protocol involves a handshake process, during which the browser and the server communicate using the HTTP protocol. In Workerman, you can intervene in the handshake process as follows.

**When workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // You can verify the legitimacy of the connection here and close it if it is not valid
        // $_SERVER['HTTP_ORIGIN'] indicates the site from which the page initiated the websocket connection
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }

        // $_GET and $_SERVER are available in onWebSocketConnect
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**When workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## Transmission of Binary Data in WebSocket Protocol

By default, the WebSocket protocol can only transmit UTF-8 text. If you want to transmit binary data, please read the following section.

In the WebSocket protocol, a flag is used in the protocol header to indicate whether the transmission is binary data or UTF-8 text data. The browser will verify the flag and the content type being transmitted. If they do not match, the connection will be terminated with an error.

Therefore, when the server sends data, the flag for the transmission data type needs to be set. In Workerman, if it is regular UTF-8 text, it needs to be set as follows (usually, this value is set by default, so manual setting is not necessary):
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

If it is binary data, it needs to be set as follows:
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Note**: If $connection->websocketType is not set, it defaults to BINARY_TYPE_BLOB (which is the UTF-8 text type). Most applications transmit UTF-8 text, such as JSON data, so manual setting of $connection->websocketType is not necessary. This property should only be set to BINARY_TYPE_ARRAYBUFFER when transmitting binary data (e.g., image data, protocol buffer data, etc.).

## Using Workerman as a WebSocket Client
You can use the [AsyncTcpConnection class](../async-tcp-connection.md) in conjunction with the [ws protocol](about-ws.md) to make Workerman act as a WebSocket client, connecting to a remote WebSocket server to achieve bi-directional real-time communication.
