# onWorkerReload
Requirement: (workerman >= 3.2.5)

## Description
```php
callback Worker::$onWorkerReload
```
This feature is not commonly used.

Sets the callback function to be executed when the Worker receives a reload signal.

The onWorkerReload callback can be used to perform many tasks, such as reloading business configuration files without the need to restart the process.

**Note**:

The default action for child processes upon receiving a reload signal is to exit and restart, so that the new process can reload the business code to complete the code update. Therefore, it is normal for the child process to exit immediately after executing the onWorkerReload callback.

If you only want the child process to execute onWorkerReload after receiving the reload signal, and do not want to exit, you can set the reloadable property of the corresponding Worker instance to false when initializing the Worker instance.


## Callback function parameters

``` $worker ```

The Worker object itself


## Example

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Set reloadable to false, so that the child process does not restart upon receiving the reload signal
$worker->reloadable = false;
// Notify all clients that the server has reloaded after a reload
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Run the worker
Worker::runAll();
```

Tip: In addition to using an anonymous function as a callback, you can also use [other callback writing methods](../faq/callback_methods.md) as a reference.
