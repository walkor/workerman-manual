# maxSendBufferSize
## 说明:
```php
int Connection::$maxSendBufferSize
```

每个连接都有一个单独的应用层发送缓冲区，如果客户端接收速度小于服务端发送速度，数据会在应用层缓冲区暂存等待发送。

此属性用来设置当前连接的应用层发送缓冲区大小。不设置默认为[Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md)(1MB)。

此属性影响[onBufferFull](../worker/on-buffer-full.md)回调。


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 设置当前连接的应用层发送缓冲区大小为102400字节
    $connection->maxSendBufferSize = 102400;
};
// 运行worker
Worker::runAll();
```
