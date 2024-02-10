# onClose
## Description:
```php
callback Connection::$onClose
```

This callback has the same function as [Worker::$onClose](../worker/on-close.md) callback, but it is only valid for the current connection. That means it can set the onClose callback for a specific connection.

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Triggered when a connection event occurs
$worker->onConnect = function(TcpConnection $connection)
{
    // Set the onClose callback for the connection
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connection closed\n";
    };
};
// Run the worker
Worker::runAll();
```

The above code is equivalent to the following

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Set the onClose callback for all connections
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// Run the worker
Worker::runAll();
```
