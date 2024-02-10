# onMessage
## Description:
```php
callback Connection::$onMessage
```


It has the same function as the [Worker::$onMessage](../worker/on-message.md) callback, but it is only valid for the current connection. In other words, it can set the onMessage callback for a specific connection.


## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// When a client connects
$worker->onConnect = function(TcpConnection $connection)
{
    // Set the onMessage callback for the connection
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('receive success');
    };
};
// Run the worker
Worker::runAll();
```

The above code has the same effect as the following:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Set the onMessage callback for all connections directly
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// Run the worker
Worker::runAll();
```
