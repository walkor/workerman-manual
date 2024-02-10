# onWorkerStart
## Description:
```php
callback Worker::$onWorkerStart
```

Sets the callback function for when a Worker sub-process starts, which gets executed each time a sub-process starts.

Note: onWorkerStart runs when a child process starts, if multiple child processes (`$worker->count > 1`) are enabled, the callback will run `$worker->count` times in total.


## Callback Function Parameters

 ``` $worker ```

Refers to the Worker object.


## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Worker starting...\n";
};
// Run the worker
Worker::runAll();
```

Note: Apart from using anonymous functions as callbacks, you can also refer to [here](../faq/callback_methods.md) for other callback styles.
