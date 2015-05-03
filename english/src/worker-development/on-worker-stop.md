# onWorkerStop
## Description:
```php
callback Worker::$onWorkerStop
```

Emitted when a Woker process stop.

## Parameters

``` $worker ```

The worker.

## Examples

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStop = function($worker)
{
    echo "Worker stopping...\n";
};
```
