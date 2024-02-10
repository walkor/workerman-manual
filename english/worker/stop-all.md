# stopAll

```php
void Worker::stopAll(void)
```

Stops the current process and exits.

> **Note**
> `Worker::stopAll()` is used to stop the current process. After the current process exits, the main process will immediately relaunch a new process. If you want to stop the entire Workerman service, please call `posix_kill(posix_getppid(), SIGINT)`.

### Parameters
None



### Return Value
None

## Example: max_request

In the following example, the child process executes `stopAll` to exit after processing 1000 requests, in order to restart a completely new process. This is similar to the `max_request` property in php-fpm and is mainly used to solve memory leak issues caused by bugs in business logic code.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Each process can execute a maximum of 1000 requests
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Number of requests processed
    static $request_count = 0;

    $connection->send('hello http');
    // If the request count reaches 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Exit the current process, and the main process will immediately start a completely new process to make up for it
         * Thus completing the process restart
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
