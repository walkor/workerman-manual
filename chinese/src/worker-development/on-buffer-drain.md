# onBufferDrain
## 说明:
```php
callback Worker::$onBufferDrain
```

每个连接都有一个单独的应用层发送缓冲区，缓冲区大小由```TcpConnection::$maxSendBufferSize```决定，默认值为1MB，可以手动设置更改大小，更改后会对所有连接生效。

该回调在应用层发送缓冲区数据全部发送完毕后触发。一般与onBufferFull配合使用，例如在onBufferFull时停止向对端继续send数据，在onBufferDrain恢复写入数据。



## 回调函数的参数

``` $connection ```

连接对象


## 范例

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function($connection)
{
    echo "bufferFull and do not send again\n";
};
$worker->onBufferDrain = function($connection)
{
    echo "buffer drain and continue send\n";
};
// 运行worker
Worker::runAll();
```

## 参见
onBufferFull 当连接的应用层发送缓冲区满时触发
