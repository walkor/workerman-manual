# reloadable
## Description:
```php
string Worker::$reloadable
```

If the worker processes can be reloaded. Default value is ```true```. If ```$worker->reloadable = true```, the worker processes will restart when get reload signal(```SIGUSR1```).


## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// reloadable
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};

// Run all workers
Worker::runAll();
```
