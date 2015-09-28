# onClose
## Description:
```php
callback Connection::$onClose
```

Is the same as ```$worker->onClose```, but only for the current connection.

## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    $connection->onClose = function($connection)
    {
        echo "connection closed\n";
    };
};

// Run all workers
Worker::runAll();
```

Is the same as

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function($connection)
{
    echo "connection closed\n";
};

// Run all workers
Worker::runAll();
```
