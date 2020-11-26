# logFile
## Description:
```php
static string Worker::$logFile
```

This is a static property. Worker::$logFile are used to record information about the workerman framework itself, such as start, stop, and some fatal errors(if any).


## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

Worker::$daemonize = true;

Worker::$logFile = '/tmp/workerman.log';
$worker = new Worker('Text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};

// Run all workers
Worker::runAll();
```
