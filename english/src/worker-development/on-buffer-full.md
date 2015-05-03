# onBufferFull
## Description:
```php
callback Worker::$onBufferFull
```

Emitted when the send buffer becomes full.

Each connection has a send write which can be set by ```$connection->maxSendBufferSize```. Default size is the value of ```TcpConnection::$defaultMaxSendBufferSize``` which is ```1MB```.


## Parameters

``` $connection ```

The instance of Connection.


## Examples

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function($connection)
{
    echo "bufferFull and do not send again\n";
};
```

