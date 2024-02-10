# resumeRecv
## Description:
```php
void Connection::resumeRecv(void)
```

Resume receiving data for the current connection. This method is useful for upload traffic control when used in conjunction with Connection::pauseRecv.

## Parameters

No parameters


## Example

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Dynamically add a property to the connection object to store the number of requests received from the current connection
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Stop receiving data after 100 requests are received for each connection
    $limit = 100;
    if (++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // Resume receiving data after 30 seconds
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Run the worker
Worker::runAll();
```

## See also
void Connection::pauseRecv(void) Stops the corresponding connection object from receiving data
