# count

## Description:
```php
int Worker::$count
```

Set how many processes the current Worker instance will start. When not set, the default value is 1.

For instructions on how to set the number of processes, please refer to [**here**](../faq/processes-count.md).

Note: This property must be set before ```Worker::runAll();``` is called in order to take effect. This feature is not supported on Windows systems.


## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Start 8 processes, simultaneously listening on port 8484, providing services using the websocket protocol
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Run worker
Worker::runAll();
```
