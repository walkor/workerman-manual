# connections
## 说明:
```php
array Worker::$connections
```

此属性中存储了**当前进程**的所有的客户端连接，这在广播时非常有用。


## 范例

```php
use Workerman\Worker;
use Workerman\Lib\Timer;
$worker = new Worker('Text://0.0.0.0:8484');
// 进程启动时设置一个定时器，定时向所有客户端连接发送数据
$worker->onWorkerStart = function($worker)
{
    // 定时，每10秒一次
    Timer::add(10, function()use($worker)
    {
        // 遍历当前进程所有的客户端连接，发送当前服务器的时间
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// 运行worker
Worker::runAll();
```
