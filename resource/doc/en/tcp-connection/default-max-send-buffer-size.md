# defaultMaxSendBufferSize
## Description:
```php
static int Connection::$defaultMaxSendBufferSize
```

This property is a global static property used to set the default application-layer send buffer size for all connections. If not set, the default is `1MB`. `Connection::$defaultMaxSendBufferSize` can be dynamically set, and the setting will only affect new connections created thereafter.

This property affects the [onBufferFull](../worker/on-buffer-full.md) callback.

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Set the default application-layer send buffer size for all connections
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Set the application-layer send buffer size for the current connection, overriding the default value
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Run the worker
Worker::runAll();
```
