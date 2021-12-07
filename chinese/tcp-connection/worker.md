# worker
## 说明:
```php
Worker Connection::$worker
```

此属性为只读属性，即当前connection对象所属的worker实例


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// 当一个客户端发来数据时，转发给当前进程所维护的其它所有客户端
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// 运行worker
Worker::runAll();
```
