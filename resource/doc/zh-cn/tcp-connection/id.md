# id

## 说明:
```php
int Connection::$id
```

连接的id。这是一个自增的整数。

注意：workerman是多进程的，每个进程内部会维护一个自增的connection id，所以多个进程之间的connection id会有重复。
如果想要不重复的connection id 可以根据需要给connection->id重新赋值，例如加上worker->id前缀。

## 参见
[Worker的connections属性](../worker/connections.md)


## 范例


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// 运行worker
Worker::runAll();
```

