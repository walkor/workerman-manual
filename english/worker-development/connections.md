# connections
## Description:
```php
array Worker::$connections
```

It contains all of the connections of the worker.


## Examples

```php
use Workerman\Worker;
use Workerman\Timer;
require_once './Workerman/Autoloader.php';

$worker = new Worker('Text://0.0.0.0:8484');
$worker->count = 6;

// Add a Timer to Every worker process when the worker process start
$worker->onWorkerStart = function($worker)
{
    // Timer every 10 seconds
    Timer::add(10, function()use($worker)
    {
        // Iterate over connections and send the time
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};

// Run all workers
Worker::runAll();
```

**Test**

```shell
telnet 127.0.0.1 8484
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
1430638160
1430638170
```
