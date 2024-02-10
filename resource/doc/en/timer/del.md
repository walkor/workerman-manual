```php
boolean \Workerman\Timer::del(int $timer_id)
```

Delete a timer.

### Parameters
``` timer_id ```

The id of the timer, which is an integer returned by the add interface.

### Return
boolean

### Example
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Start a specified number of processes to run the timed task, pay attention to the problem of concurrent multiple processes
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Run every 2 seconds
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // Run a one-time task after 20 seconds, and delete the timer that runs every 2 seconds
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Run the worker
Worker::runAll();
```

### Example (delete the current timer in the timer callback)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Note that when using the current timer id in the callback, you must use the reference (&) to introduce it
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // After running 10 times, delete the timer
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Run the worker
Worker::runAll();
```
