# user

## Description:
```php
string Worker::$user
```

Set the user of the worker processes. This needs appropriate privileges (usually root) on the system to be able to perform this.

Recommend ```www-data```, ```apache```, ```nobody``` and so on.


## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Set the user of worker processes
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};

// Run all workers
Worker::runAll();
```
