# onBufferDrain
## Description:
```php
callback Worker::$onBufferDrain
```

Emitted when the send buffer becomes empty.


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
$worker->onBufferDrain = function($connection)
{
    echo "buffer drain and continue send\n";
};
```
