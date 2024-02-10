# reloadable
## Description:
```php
bool Worker::$reloadable
```

When executing `php start.php reload`, a reload signal (SIGUSR1) will be sent to all child processes.

After receiving the reload signal, the child process will automatically exit and the main process will automatically start a new process, typically used to update business code.

When $reloadable of the process is set to false, receiving the reload signal will only trigger [onWorkerReload](on-worker-reload.md) and will not restart the current process.

For example, in the Gateway/Worker model, the gateway process is responsible for maintaining client connections, while the worker process is responsible for handling requests. Setting the gateway process's reloadable attribute to false allows updating business code without disconnecting client connections during reload.

## Example
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Set whether this instance will restart after receiving a reload signal
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Run the worker
Worker::runAll();
```
