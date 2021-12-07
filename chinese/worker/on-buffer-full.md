# onBufferFull
## 说明:
```php
callback Worker::$onBufferFull
```

每个连接都有一个单独的应用层发送缓冲区，如果客户端接收速度小于服务端发送速度，数据会在应用层缓冲区暂存，如果缓冲区满则会触发onBufferFull回调。

缓冲区大为[TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)，默认值为1MB，可以为当前连接动态设置缓冲区大小例如：
```php
// 设置当前连接发送缓冲区，单位字节
$connection->maxSendBufferSize = 102400;
```
也可以利用[TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) 设置所有连接默认缓冲区的大小，例如代码：
```php
use Workerman\Connection\TcpConnection;
// 设置所有连接的默认应用层发送缓冲区大小，单位字节
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

该回调**可能**会在调用Connection::send后立刻被触发，比如发送大数据或者连续快速的向对端发送数据，由于网络等原因数据被大量积压在对应连接的发送缓冲区，当超过```TcpConnection::$maxSendBufferSize```上限时触发。

当发生onBufferFull事件时，开发者一般需要采取措施，例如停止向对端发送数据，等待发送缓冲区的数据被发送完毕(onBufferDrain事件)等。

当调用Connection::send(`$A`)后导致触发onBufferFull时，不管本次send的数据`$A`多大，即使大于`TcpConnection::$maxSendBufferSize`，本次要发送的数据仍然会被放入发送缓冲区。也就是说发送缓冲区实际放入的数据可能远远大于`TcpConnection::$maxSendBufferSize`，当发送缓冲区的数据已经大于`TcpConnection::$maxSendBufferSize`时，仍然继续Connection::send(`$B`)数据，则这次send的`$B`数据不会放入发送缓冲区，而是被丢弃掉，并触发`onError`回调。

总结来说，只要发送缓冲区还没满，哪怕只有一个字节的空间，调用Connection::send(```$A```)肯定会把```$A```放入发送缓冲区，如果放入发送缓冲区后，发送缓冲区大小超过了```TcpConnection::$maxSendBufferSize```限制，则会触发onBufferFull回调。


## 回调函数的参数

 ``` $connection ```

连接对象，即[TcpConnection实例](../tcp-connection.md)，用于操作客户端连接，如[发送数据](../tcp-connection/send.md)，[关闭连接](../tcp-connection/close.md)等


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
// 运行worker
Worker::runAll();
```

提示：除了使用匿名函数作为回调，还可以[参考这里](../faq/callback_methods.md)使用其它回调写法。

## 参见
onBufferDrain 当连接的应用层发送缓冲区数据全部发送完毕时触发

