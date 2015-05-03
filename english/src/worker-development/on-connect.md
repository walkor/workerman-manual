# onConnect
## Description:
```php
callback Worker::$onConnect
```

Emitted when a new connection is made.

## Parameters

``` $connection ```

The instance of Connection.


## Examples

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
```
