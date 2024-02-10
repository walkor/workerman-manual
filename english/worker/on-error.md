# onError
## Description:
```php
callback Worker::$onError
```

This event is triggered when an error occurs on the client's connection.

Currently, the error types include:

1. Failed due to a client connection being closed when calling Connection::send (followed by triggering the onClose callback)
   ``` 
   (code:WORKERMAN_SEND_FAIL msg:client closed)
   ```

2. After triggering onBufferFull (when the send buffer is full), calling Connection::send again and the send buffer is still full, resulting in a send failure (onClose callback will not be triggered)
   ``` 
   (code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)
   ```

3. When the AsyncTcpConnection asynchronous connection fails (followed by triggering the onClose callback)
   ``` 
   (code:WORKERMAN_CONNECT_FAIL msg:error message returned by stream_socket_client)
   ```

## Callback function parameters

``` $connection ```

The connection object, that is, the [TcpConnection instance](../tcp-connection.md), used to operate the client connection, such as [sending data](../tcp-connection/send.md), [closing the connection](../tcp-connection/close.md), etc.

``` $code ```

Error code

``` $msg ```

Error message

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// Run the worker
Worker::runAll();
```

Note: In addition to using anonymous functions as callbacks, other callback methods can also be [referenced here](../faq/callback_methods.md).
