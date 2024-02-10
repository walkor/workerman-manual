# How to Broadcast Data

## Example (Scheduled Broadcasting)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// The number of processes must be 1 in this example
$worker->count = 1;
// Set a timer when the process starts to send data to all client connections regularly
$worker->onWorkerStart = function($worker)
{
    // Send data to all client connections every 10 seconds
    Timer::add(10, function() use ($worker)
    {
        // Iterate through all client connections of the current process and send the current server time
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Run the worker
Worker::runAll();
```

## Example (Group Chat)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// The number of processes must be 1 in this example
$worker->count = 1;
// Broadcast messages from clients to other users
$worker->onMessage = function(TcpConnection $connection, $message) use ($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Run the worker
Worker::runAll();
```

## Explanation:
**Single Process:**
The examples above can only work with a **single process** (```$worker->count=1```). This is because in a multi-process scenario, multiple clients may connect to different processes, and the client connections between processes are isolated and unable to communicate directly. This means that the client connection objects in process A cannot **directly** send data to process B. To achieve this, inter-process communication is required, such as using the Channel component, for example the [Cluster Sending Example](../components/channel-examples.md) or the [Group Sending Example](../components/channel-examples2.md).

**Suggestion to use GatewayWorker:**
The GatewayWoker framework, which is developed based on Workerman, provides a more convenient push mechanism, including multicast, broadcast, etc. It can be deployed with multiple processes or even multiple servers. If there is a need to push data to clients, it is recommended to use the GatewayWorker framework.

GatewayWorker manual address: https://doc2.workerman.net
