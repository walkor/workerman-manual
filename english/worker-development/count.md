# count

## Description :
```php
int Worker::$count
```

Set the process count of the worker instance. Default value is ```1```.


## Examples


```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 8 porcesses
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};

// Run all workers
Worker::runAll();
```
