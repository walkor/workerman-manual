# onMessage
## 说明:
```php
callback Connection::$onMessage
```


作用与[Worker::$onMessage](../worker/on-message.md)回调相同，区别是只针对当前连接有效，也就是可以针对某个连接的设置onMessage回调。


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 当有客户端连接事件时
$worker->onConnect = function(TcpConnection $connection)
{
    // 设置连接的onMessage回调
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('receive success');
    };
};
// 运行worker
Worker::runAll();
```

上面代码与下面的效果是一样的

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 直接设置所有连接的onMessage回调
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// 运行worker
Worker::runAll();
```
