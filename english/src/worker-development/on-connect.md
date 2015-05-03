# onConnect
## Description:
```php
callback Worker::$onConnect
```

Emitted when a new connection is made.

## Parameters

``` $connection ```

The instance of Connection.


## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};

// Run all workers
Worker::runAll();
```
