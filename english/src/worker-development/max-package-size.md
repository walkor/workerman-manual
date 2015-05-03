# maxPackageSize

## Description:
```php
static int Connection::$maxPackageSize
```

This is a static property. Set the max package size can be received .Default value is 10MB.


## Examples


```php
use Workerman\Worker;
use Workerman\Protocols\TcpConnection;

// 设置每个连接接收的数据包最大为1024000字节
TcpConnection::$maxPackageSize = 1024000;

$worker = new Worker('Websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    $connection->send('hello');
};
```
