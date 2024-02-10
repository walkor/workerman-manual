# name

## Description:
```php
string Worker::$name
```

Set the name of the current Worker instance to identify the process when running the status command. If not set, the default is none.

## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Set the name of the instance
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Run the worker
Worker::runAll();
```
