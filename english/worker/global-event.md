# globalEvent

## Description:
```php
static Event Worker::$globalEvent
```

This property is a global static property, which is a global event loop instance, and can be used to register read/write events or signal events for file descriptors.


## Example

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // When the process receives the SIGALRM signal, output some information
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// Run the worker
Worker::runAll();
```

## Test
After Workerman starts, it will output the current process ID (a number). Run the following command in the terminal:
```
kill -SIGALRM process_id
```
The server will then output:
```
Get signal SIGALRM
```
