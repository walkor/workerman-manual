# daemonize
## Description
```php
static bool Worker::$daemonize
```

This attribute is a global static attribute, indicating whether to run in daemon mode. If the startup command uses the `-d` parameter, this attribute will be automatically set to true. It can also be set manually in the code.

Note: This attribute must be set before `Worker::runAll();` is executed to be effective. This feature is not supported on Windows systems.

## Example
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Run the worker
Worker::runAll();
```
