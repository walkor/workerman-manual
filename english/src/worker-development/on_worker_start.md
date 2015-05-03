# onWorkerStart
## Description:
```php
callback Worker::$onWorkerStart
```

Emitted when a Woker process start.


## Parameters

``` $worker ```

The worker.



## Examples


```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
```
