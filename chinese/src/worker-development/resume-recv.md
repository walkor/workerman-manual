# resumeRecv
## 说明:
```php
void Connection::resumeRecv(void)
```

使当前连接继续接收数据。此方法与Connection::pauseRecv配合使用，对于上传流量控制非常有用

## 参数

无参数


## 范例

```php
use Workerman\Worker;
use Workerman\Lib\Timer;
require_once './Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // 给connection对象动态添加一个属性，用来保存当前连接发来多少个请求
    $connection->messageCount = 0;
};
$worker->onMessage = function($connection, $data)
{
    // 每个连接接收100个请求后就不再接收数据
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // 30秒后恢复接收数据
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// 运行worker
Worker::runAll();
```

## 参见
void Connection::pauseRecv(void) 使得对应连接对象停止接收数据
