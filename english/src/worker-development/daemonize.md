# daemonize
## Description:
```php
static bool Worker::$daemonize
```

This is a static property. Workerman will run as daemon mode when  ```Worker::$daemonize=true``` , otherwise run as debug mode by default. Use ```-d``` option start workerman will run as daemon mode.


## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

Worker::$daemonize = true;
$worker = new Worker('Text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};

// Run all workers
Worker::runAll();
```
