# getRemotePort
## Description:
```php
int Connection::getRemotePort()
```

Get remote port

## Parameters

This function has no parameters.


## Return Values

The remote port. Example ```5698```.

## Examples

```php
use Workerman\Worker;
$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo "new connection from address " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
```
