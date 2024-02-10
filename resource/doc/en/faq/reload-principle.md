# Principle of Smooth Restart
## What is Smooth Restart?

Smooth restart is different from regular restart. It allows for restarting services (typically short-link businesses) without affecting users, in order to reload PHP programs and update business code.

Smooth restart is generally applied during business updates or version releases, and it can prevent temporary unavailability of services caused by code deployment and service restart.

> **Note**
> Windows systems do not support reload.

> **Note**
> For long-lived connections (such as WebSockets), when a process undergoes a smooth restart, the connections will be disconnected. The solution is to use an architecture similar to [gatewayWorker](https://www.workerman.net/doc/gateway-worker), where a group of processes is dedicated to maintaining connections, and the `reloadable` property of these processes is set to `false`. Another group of worker processes handle the business logic, and communication between the gateway and worker processes is achieved through TCP. When the business logic needs to be changed, only the worker processes need to be restarted.

## Limitations
**Note: Only the files loaded in the on{...} callback will be automatically updated after a smooth restart. The files directly loaded or hardcoded in the startup script will not be automatically updated upon reload.**

#### The following code will not be updated after reload
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // Hardcoded code does not support hot updates
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // The file directly loaded in the startup script does not support hot updates
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Assuming MessageHandler class has an onMessage method
```


#### The following code will be automatically updated after reload
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart is a callback triggered after the process starts
    require_once __DIR__ . '/your/path/MessageHandler.php'; // The file loaded after the process starts supports hot updates
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
After making changes to MessageHandler.php, executing `php start.php reload` will reload it into memory, achieving the update of business logic.

> **Note**
> The 'require_once' statement is used in the above code for demonstration purposes. If your project supports PSR-4 autoloading, the 'require_once' statement is not needed.

## Principle of Smooth Restart

Workerman is divided into a main process and child processes. The main process is responsible for monitoring the child processes, while the child processes handle client connections, incoming request data, and corresponding processing and response to clients. When the business code is updated, we only need to update the child processes to achieve the purpose of code update.

When the Workerman main process receives a smooth restart signal, it will send a graceful shutdown signal to one of the child processes (allowing the corresponding process to finish the current request before exiting). After this process exits, the main process will create a new child process (which loads the new PHP code), and then the main process will send a stop command to another old process, and this process will be restarted, and this process will continue until all old processes are replaced.

We can see that the smooth restart is actually achieved by allowing the old business processes to exit one by one and then creating new processes one by one. To avoid affecting users during a smooth restart, it is required that the processes do not retain user-related state information. Business processes are best to be stateless, to avoid information loss due to process exits.
