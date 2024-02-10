# onClose
## Descripción:
```php
callback Worker::$onClose
```

Callback function triggered when a client connection is disconnected from Workerman. Regardless of how the connection is disconnected, the `onClose` callback will be triggered. Each connection will only trigger `onClose` once.

Note: If the other end disconnects due to extreme conditions such as network disconnection or power failure, Workerman cannot timely send a FIN packet to the TCP, so it cannot know that the connection has been disconnected, and the `onClose` cannot be triggered in time. This situation needs to be resolved through application layer heartbeats. The implementation of connection heartbeat in Workerman can be found [here](../faq/heartbeat.md). If using the GatewayWorker framework, simply use the GatewayWorker framework's heartbeat mechanism, see [here](https://doc2.workerman.net/heartbeat.html).

Since UDP is connectionless, using UDP will not trigger the `onConnect` callback, nor will it trigger the `onClose` callback.

## Parámetros de la función de devolución de llamada:

 ``` $connection ```

Objeto de conexión, es decir, una instancia de [TcpConnection](../tcp-connection.md), que se utiliza para operar la conexión del cliente, como [enviar datos](../tcp-connection/send.md), [cerrar la conexión](../tcp-connection/close.md), entre otras.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "conexión cerrada\n";
};
// Ejecutar el worker
Worker::runAll();
```

Nota: Aparte de usar una función anónima como devolución de llamada, también se pueden utilizar otras maneras de escribir devoluciones de llamada, consulte [aquí](../faq/callback_methods.md).
