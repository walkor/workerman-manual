# onError
## 说明:
```php
callback Worker::$onError
```

当客户端的连接上发生错误时触发。

目前错误类型有

1、调用Connection::send由于客户端连接断开导致的失败（紧接着会触发onClose回调） ```
(code:WORKERMAN_SEND_FAIL msg:client closed)```


2、在触发onBufferFull后(发送缓冲区已满)，仍然调用Connection::send，并且发送缓冲区仍然是满的状态导致发送失败(不会触发onClose回调)```
(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```


3、使用AsyncTcpConnection异步连接失败时(紧接着会触发onClose回调) ```
(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client返回的错误消息)```


## 回调函数的参数

 ``` $connection ```

连接对象，即[TcpConnection实例](../tcp-connection.md)，用于操作客户端连接，如[发送数据](../tcp-connection/send.md)，[关闭连接](../tcp-connection/close.md)等

 ``` $code ```

错误码

 ``` $msg ```

错误消息


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// 运行worker
Worker::runAll();
```

提示：除了使用匿名函数作为回调，还可以[参考这里](../faq/callback_methods.md)使用其它回调写法。