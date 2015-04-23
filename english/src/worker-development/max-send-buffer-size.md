# maxSendBufferSize
## 说明:
```php
static int Connection::$maxSendBufferSize
```

此属性为全局静态属性，用来设置每个客户端连接的应用层发送缓冲区大小。不设置默认为1MB。

此属性影响onBuffer回调


## 范例


```php
use Workerman\Worker;
use Workerman\Protocols\TcpConnection;

// 设置每个连接的应用层发送缓冲区大小为102400字节
TcpConnection::$maxSendBufferSize = 102400;

$worker = new Worker('Websocket://0.0.0.0:8484');
$worker->onMessage = function($connection, $data)
{
    $connection->send('hello');
};
```
