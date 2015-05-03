# maxSendBufferSize
## Description:
```php
int Connection::$maxSendBufferSize
```

Set the send buffer size of the connection.


## Examples


```php
use Workerman\Worker;

$worker = new Worker('Websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    $connection->maxSendBufferSize = 2*1024*1024;
};
```
