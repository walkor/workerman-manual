# maxSendBufferSize
## Description:
```php
int Connection::$maxSendBufferSize
```

Each connection has a separate application-layer send buffer. If the client's receiving speed is slower than the server's sending speed, data will be temporarily stored in the application-layer buffer, waiting to be sent.

This property is used to set the size of the application-layer send buffer for the current connection. If not set, the default is [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1MB).

This property affects the [onBufferFull](../worker/on-buffer-full.md) callback.


## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Set the application-layer send buffer size for the current connection to 102400 bytes
    $connection->maxSendBufferSize = 102400;
};
// Run the worker
Worker::runAll();
```
