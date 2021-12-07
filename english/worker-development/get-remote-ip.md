# getRemoteIp
## Description:
```php
string Connection::getRemoteIp()
```

get remote ip

## Parameters

This function has no parameters.

## Return Values
The remote ip. For Example ```111.26.36.12```.

## Examples

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};

// Run all workers
Worker::runAll();
```
