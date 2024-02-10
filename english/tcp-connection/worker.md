# worker
## Description:
```php
Worker Connection::$worker
```

This property is a read-only property, which refers to the worker instance to which the current connection object belongs.


## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// When a client sends data, forward it to all other clients maintained by the current process
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Run the worker
Worker::runAll();
```
