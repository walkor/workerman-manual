# pauseRecv
## Description:
```php
void Connection::pauseRecv(void)
```

Stops the current connection from receiving data. The onMessage callback of this connection will not be triggered. This method is very useful for traffic control for uploads.

## Parameters

No parameters

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Dynamically add a property to the connection object to store the number of requests received from the current connection
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Stop receiving data after 100 requests are received for each connection
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Run the worker
Worker::runAll();
```

## See also
void Connection::resumeRecv(void) Resumes the data reception for the corresponding connection object
