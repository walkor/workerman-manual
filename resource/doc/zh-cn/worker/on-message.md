# onMessage
## 说明:
```php
callback Worker::$onMessage
```

当客户端通过连接发来数据时(Workerman收到数据时)触发的回调函数

## 回调函数的参数

 ``` $connection ```

连接对象，即[TcpConnection实例](../tcp-connection.md)，用于操作客户端连接，如[发送数据](../tcp-connection/send.md)，[关闭连接](../tcp-connection/close.md)等

 ``` $data ```

客户端连接上发来的数据，如果Worker指定了协议，则$data是对应协议decode（解码）了的数据。数据类型与协议`decode()`实现有关，`websocket` `text` `frame` 为字符串，HTTP协议为 [`Workerman\Protocols\Http\Request`](../http/request.md)对象。


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// 运行worker
Worker::runAll();
```

提示：除了使用匿名函数作为回调，还可以[参考这里](../faq/callback_methods.md)使用其它回调写法。