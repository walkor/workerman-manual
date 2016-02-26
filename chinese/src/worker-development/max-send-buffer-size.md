# maxSendBufferSize
## 说明:
```php
int Connection::$maxSendBufferSize
```

此属性用来设置当前连接的应用层发送缓冲区大小。不设置默认为```Connection::$defaultMaxSendBufferSize```(1MB)。```Connection::$maxSendBufferSize``` 和 ```Connection::$defaultMaxSendBufferSize```均可以动态设置。

此属性影响onBufferFull回调


## 范例

```php
use Workerman\Worker;
use Workerman\Protocols\TcpConnection;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // 设置当前连接的应用层发送缓冲区大小为102400字节
    $connection->maxSendBufferSize = 102400;
};
// 运行worker
Worker::runAll();
```
