# worker
## Description:
```php
Worker Connection::$worker
```

Read only. The owner of the connection.


## Examples


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('Websocket://0.0.0.0:8484');

// When received message，forwarded to all connections of the worker
$worker->onMessage = function($connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};

// Run all workers
Worker::runAll();
```
