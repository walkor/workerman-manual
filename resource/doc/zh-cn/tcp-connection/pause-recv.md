# pauseRecv
## 说明:
```php
void Connection::pauseRecv(void)
```

使当前连接停止接收数据。该连接的onMessage回调将不会被触发。此方法对于上传流量控制非常有用

## 参数

无参数


## 范例

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // 给connection对象动态添加一个属性，用来保存当前连接发来多少个请求
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // 每个连接接收100个请求后就不再接收数据
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// 运行worker
Worker::runAll();
```

## 参见
void Connection::resumeRecv(void) 使得对应连接对象恢复接收数据
