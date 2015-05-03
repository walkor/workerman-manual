# globalEvent

## Description:
```php
static Event Worker::$globalEvent
```

This is a static property. ```Worker::$globalEvent``` is the global eventloop.


## Examples

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once './Workerman/Autoloader.php';

$worker = new Worker('Text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    //  Install SIGALRM handler
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};

// Run all workers
Worker::runAll();
```
