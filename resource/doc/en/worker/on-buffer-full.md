# onBufferFull
## Description:
```php
callback Worker::$onBufferFull
```

Each connection has a separate application layer send buffer. If the client's receiving speed is slower than the server's sending speed, the data will be stored in the application layer buffer. If the buffer is full, the onBufferFull callback will be triggered.

The size of the buffer is [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md), with a default value of 1MB. The buffer size can be dynamically set for the current connection, for example:
```php
// Set the maximum send buffer size for the current connection in bytes
$connection->maxSendBufferSize = 102400;
```
Alternatively, you can use [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) to set the default buffer size for all connections, for example:
```php
use Workerman\Connection\TcpConnection;
// Set the default application layer send buffer size for all connections in bytes
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

This callback **may** be triggered immediately after calling Connection::send, for example when sending large amounts of data or rapidly sending data to the other end. Due to network or other reasons, data may accumulate in the send buffer of the corresponding connection and trigger the callback when it exceeds the ```TcpConnection::$maxSendBufferSize``` limit.

When the onBufferFull event occurs, developers generally need to take action, such as stopping sending data to the other end and waiting for the data in the send buffer to be sent out (onBufferDrain event).

When calling Connection::send(`$A`) results in triggering onBufferFull, regardless of the size of the data `$A` being sent, even if it exceeds `TcpConnection::$maxSendBufferSize`, the data to be sent will still be placed in the send buffer. In other words, the actual data placed in the send buffer may be much larger than `TcpConnection::$maxSendBufferSize`. When the data in the send buffer reaches or exceeds `TcpConnection::$maxSendBufferSize`, calling Connection::send(`$B`) will not place the data `$B` in the send buffer, but will instead be discarded and trigger the `onError` callback.

In summary, as long as the send buffer is not full, even if there is only one byte of space left, calling Connection::send(`$A`) will definitely place `$A` in the send buffer. If the send buffer size exceeds the `TcpConnection::$maxSendBufferSize` limit after placing it in the send buffer, the onBufferFull callback will be triggered.


## Parameters of the Callback Function

 ``` $connection ```

The connection object, i.e. [TcpConnection instance](../tcp-connection.md), used for operating the client connection, such as [sending data](../tcp-connection/send.md), [closing the connection](../tcp-connection/close.md), etc.


## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
// Run the worker
Worker::runAll();
```

Note: In addition to using anonymous functions as callbacks, you can also [refer to here](../faq/callback_methods.md) for other callback writing methods.


## See Also
onBufferDrain Triggered when all the data in the application layer send buffer of a connection has been sent out
