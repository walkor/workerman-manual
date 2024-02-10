# onClose
## Description:
```php
callback Worker::$onClose
```

Callback function triggered when the client connection is disconnected from Workerman. Regardless of how the connection is terminated, the `onClose` event will be triggered. Each connection will only trigger `onClose` once.

Note: If the other end disconnects due to extreme conditions such as network disconnection or power failure, Workerman cannot know that the connection has been disconnected in time and cannot trigger `onClose` in time because it cannot send a TCP fin packet to Workerman. This situation needs to be resolved through application-layer heartbeats. The implementation of connection heartbeat in Workerman can be found [here](../faq/heartbeat.md). If using the GatewayWorker framework, you can directly use the GatewayWorker framework's heartbeat mechanism, as described [here](https://doc2.workerman.net/heartbeat.html).

Since UDP is connectionless, when using UDP, the `onConnect` callback will not be triggered, nor will the `onClose` callback.

## Callback Function Parameters

``` $connection ```

The connection object, i.e., [TcpConnection instance](../tcp-connection.md), used to manipulate client connections, such as [sending data](../tcp-connection/send.md), [closing the connection](../tcp-connection/close.md), etc.

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// Run the worker
Worker::runAll();
```

Note: In addition to using anonymous functions as callbacks, other callback writing methods can be found in the [documentation](../faq/callback_methods.md).
