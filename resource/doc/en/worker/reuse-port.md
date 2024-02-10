# reusePort
> **Notice**
> Requires workerman>=3.2.1 PHP>=7.0. This feature is not supported on Windows and Mac OS.

## Description:

```php
bool Worker::$reusePort
```

Set whether the current worker enables the reuse of listening ports (SO_REUSEPORT option for sockets).

Enabling port reuse allows multiple unrelated processes to listen on the same port, with the system kernel handling load balancing and deciding which process to handle the socket connection. This prevents the thundering herd effect and can improve the performance of multi-process short connection applications.

**Note:** This feature requires PHP version >=7.0.

## Example 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Run worker
Worker::runAll();
```

## Example 2: Workerman listen on multiple ports (multiple protocols)
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Add a new listener in each process after it is started
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Multiple processes listen on the same port (inherited sockets not from parent processes)
     * Need to enable port reuse, otherwise Address already in use error will occur
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Start listening
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Run worker
Worker::runAll();
```
