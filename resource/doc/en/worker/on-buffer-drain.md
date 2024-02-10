# onBufferDrain
## Description:
```php
callback Worker::$onBufferDrain
```
Each connection has a separate application-layer send buffer, the size of which is determined by `TcpConnection::$maxSendBufferSize` with a default value of 1MB. It can be manually set to change the size, which will take effect for all connections after the change.

This callback is triggered after all the data in the application-layer send buffer has been sent. It is generally used in conjunction with `onBufferFull`, for example, pausing sending data to the other end when `onBufferFull` is triggered, and resuming writing data when `onBufferDrain` is triggered.

## Callback Function Parameters
``` $connection ```
The connection object, which is a [TcpConnection instance](../tcp-connection.md), used to operate the client connection, such as [sending data](../tcp-connection/send.md), [closing the connection](../tcp-connection/close.md), and so on.

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
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "buffer drain and continue send\n";
};
// Run the worker
Worker::runAll();
```
Note: In addition to using an anonymous function as a callback, other callback methods can also be used [reference here](../faq/callback_methods.md).

## See Also
onBufferFull Triggered when the application-layer send buffer of the connection is full
