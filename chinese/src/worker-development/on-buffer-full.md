# onBufferFull
## 说明:
```php
callback Worker::$onBufferFull
```

每个连接都有一个单独的应用层发送缓冲区，缓冲区大小由```TcpConnection::$maxSendBufferSize```决定，默认值为1MB，可以手动设置更改大小，更改后会对所有连接生效。

该回调**可能**会在调用Connection::send后立刻被触发，比如发送大数据或者连续快速的向对端发送数据，由于网络等原因数据被大量积压在对应连接的发送缓冲区，当超过```TcpConnection::$maxSendBufferSize```上限时触发。

当发生onBufferFull事件时，开发者一般需要采取措施，例如停止向对端发送数据，等待发送缓冲区的数据被发送完毕(onBufferDrain事件)等。

当调用Connection::send(```$A```)后导致触发onBufferFull时，不管本次send的数据```$A```多大，即使大于```TcpConnection::$maxSendBufferSize```，本次要发送的数据仍然会被放入发送缓冲区。也就是说发送缓冲区实际放入的数据可能远远大于```TcpConnection::$maxSendBufferSize```，当发送缓冲区的数据已经大于```TcpConnection::$maxSendBufferSize```时，仍然继续Connection::send(```$B```)数据，则这次send的```$B```数据不会放入发送缓冲区，而是被丢弃掉，并触发onError回调。

总结来说，只要发送缓冲区还没满，哪怕只有一个字节的空间，调用Connection::send(```$A```)肯定会把```$A```放入发送缓冲区，如果放入发送缓冲区后，发送缓冲区大小超过了```TcpConnection::$maxSendBufferSize```限制，则会触发onBufferFull回调。


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
// 运行worker
Worker::runAll();
```

## 参见
onBufferDrain 当连接的应用层发送缓冲区数据全部发送完毕时触发
