# onConnect
## Description
```php
callback Worker::$onConnect
```

The ```onConnect``` callback function is triggered when the client establishes a connection with Workerman (after the completion of the TCP three-way handshake). The ```onConnect``` callback is triggered only once for each connection.

Note: The ```onConnect``` event only represents the completion of the TCP three-way handshake between the client and Workerman. At this point, the client has not sent any data. Apart from obtaining the remote IP address using ```$connection->getRemoteIp()```, there is no other data or information to identify the client. Therefore, it is not possible to ascertain the identity of the client within the onConnect event. To identify the client, the client needs to send authentication data, such as a token or username/password, which can be authenticated in the [onMessage callback](on-message.md).

Since UDP is connectionless, using UDP will not trigger the onConnect callback or the onClose callback.

## Callback Function Parameters
``` $connection ```

The connection object, i.e., [TcpConnection instance](../tcp-connection.md), used for operating the client connection, such as [sending data](../tcp-connection/send.md) or [closing the connection](../tcp-connection/close.md).

## Example
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// Run the worker
Worker::runAll();
```

Note: Apart from using anonymous functions as callbacks, other callback writing methods can be found [here](../faq/callback_methods.md).
