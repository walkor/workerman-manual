# del
## Description:
```php
boolean \Workerman\Timer::del(int $timer_id)
```
Stops a Timer.

### Parameters
``` timer_id ```

TimerId which returned by ```Timer::add```

### Return Values
boolean


### Examples

```php
use \Workerman\Worker;
require_once './Workerman/Autoloader.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    $timer_id = \Workerman\Timer::add(2, function()
    {
        echo "task run\n";
    });
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Run all workers
Worker::runAll();
```
