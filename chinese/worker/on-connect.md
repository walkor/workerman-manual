# onConnect
## 说明:
```php
callback Worker::$onConnect
```

当客户端与Workerman建立连接时(TCP三次握手完成后)触发的回调函数。每个连接只会触发一次```onConnect```回调。

注意：onConnect事件仅仅代表客户端与Workerman完成了TCP三次握手，这时客户端还没有发来任何数据，此时除了通过```$connection->getRemoteIp()```获得对方ip，没有其他可以鉴别客户端的数据或者信息，所以在onConnect事件里无法确认对方是谁。要想知道对方是谁，需要客户端发送鉴权数据，例如某个token或者用户名密码之类，在[onMessage回调](on-message.md)里做鉴权。

由于udp是无连接的，所以当使用udp时不会触发onConnect回调，也不会触发onClose回调。

## 回调函数的参数

 ``` $connection ```

连接对象，即[TcpConnection实例](../tcp-connection.md)，用于操作客户端连接，如[发送数据](../tcp-connection/send.md)，[关闭连接](../tcp-connection/close.md)等


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// 运行worker
Worker::runAll();
```

提示：除了使用匿名函数作为回调，还可以[参考这里](../faq/callback_methods.md)使用其它回调写法。