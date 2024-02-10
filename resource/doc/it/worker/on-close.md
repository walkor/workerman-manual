# onClose
## Descrizione:
```php
callback Worker::$onClose
```

Callback function triggered when the client connection is disconnected from Workerman. The `onClose` is triggered regardless of how the connection is disconnected. Each connection triggers `onClose` only once.

Note: If the remote end disconnects due to extreme situations such as network disconnection or power failure, Workerman cannot timely send the TCP fin packet to know that the connection has been disconnected and cannot trigger `onClose` in time. This situation needs to be addressed through the application layer heartbeat. For the implementation of the connection heartbeat in Workerman, see [here](../faq/heartbeat.md). If using the GatewayWorker framework, simply use the GatewayWorker framework's heartbeat mechanism, see [here](https://doc2.workerman.net/heartbeat.html).

Since UDP is connectionless, using UDP will not trigger the `onConnect` callback, nor will it trigger the `onClose` callback.

## Parameters of the callback function

``` $connection ```

Connection object, i.e., [TcpConnection instance](../tcp-connection.md), used to operate the client connection, such as [sending data](../tcp-connection/send.md), [closing the connection](../tcp-connection/close.md), etc.

## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function (TcpConnection $connection) {
    echo "connection closed\n";
};
// Run the worker
Worker::runAll();
```

Note: In addition to using an anonymous function as a callback, other callback writing methods can also be used [refer to here](../faq/callback_methods.md).
