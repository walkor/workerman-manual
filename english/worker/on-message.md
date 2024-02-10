# onMessage
## Description:
```php
callback Worker::$onMessage
```

Callback function triggered when the client sends data through the connection (when Workerman receives data).

## Callback function parameters
``` $connection ```

Connection object, i.e. [TcpConnection instance](../tcp-connection.md), used to operate client connections, such as [sending data](../tcp-connection/send.md), [closing the connection](../tcp-connection/close.md), etc.

``` $data ```

Data sent from the client connection. If the Worker specifies a protocol, then $data is the decoded data corresponding to the protocol. The data type depends on the implementation of the `decode()` method in the protocol. For example, for `websocket`, `text`, `frame`, the data type is a string, and for the HTTP protocol it is a [`Workerman\Protocols\Http\Request`](../http/request.md) object.

## Example
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// Run the worker
Worker::runAll();
```

Note: In addition to using an anonymous function as a callback, other [callback writing methods](../faq/callback_methods.md) can also be used.
