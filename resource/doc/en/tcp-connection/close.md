# close
## Description:
```php
void Connection::close(mixed $data = '')
```

Safely close the connection.

Calling close will wait for the data in the send buffer to be sent before closing the connection and trigger the `onClose` callback of the connection.

## Parameters

``` $data ```

Optional parameter, the data to be sent (if a protocol is specified, the protocol's encode method will automatically be called to pack the ```$data``` data), and the connection will be closed after the data is sent, and then trigger the `onClose` callback.

## Example

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Run the worker
Worker::runAll();
```
