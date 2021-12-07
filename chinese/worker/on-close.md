# onClose
## 说明:
```php
callback Worker::$onClose
```

当客户端连接与Workerman断开时触发的回调函数。不管连接是如何断开的，只要断开就会触发```onClose```。每个连接只会触发一次```onClose```。

注意：如果对端是由于断网或者断电等极端情况断开的连接，这时由于无法及时发送tcp的fin包给workerman，workerman就无法得知连接已经断开，也就无法及时触发```onClose```。这种情况需要通过应用层心跳来解决。workerman中连接的心跳实现参见[这里](../faq/heartbeat.md)。如果使用的是GatewayWorker框架，则直接使用GatewayWorker框架的心跳机制即可，参见[这里](https://doc2.workerman.net/heartbeat.html)。

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
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// 运行worker
Worker::runAll();
```

提示：除了使用匿名函数作为回调，还可以[参考这里](../faq/callback_methods.md)使用其它回调写法。