# onClose
## Description:
```php
callback Worker::$onClose
```

The callback function triggered when a client's connection is disconnected from Workerman. Regardless of how the connection is terminated, the `onClose` event will be triggered. Each connection will trigger `onClose` only once.

Note: If the connection is terminated due to extreme circumstances such as loss of network or power, Workerman may not be able to timely send a TCP fin packet to know that the connection has been terminated, and therefore cannot trigger `onClose` promptly. This kind of situation needs to be resolved by using application-layer heartbeats. For the implementation of connection heartbeats in Workerman, refer to [here](../faq/heartbeat.md). If using the GatewayWorker framework, simply utilize the heartbeat mechanism provided by the GatewayWorker framework, refer to [here](https://doc2.workerman.net/heartbeat.html).

Since UDP is connectionless, when using UDP, the `onConnect` callback will not be triggered, and the `onClose` callback will not be triggered either.

## Callback Function Parameters
``` $connection ```

Connection object, i.e. [TcpConnection instance](../tcp-connection.md), used to operate client connections such as [sending data](../tcp-connection/send.md), [closing connections](../tcp-connection/close.md), etc.

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
// Run worker
Worker::runAll();
```

Note: In addition to using an anonymous function as a callback, you can also refer to [here](../faq/callback_methods.md) for other callback writing methods.
