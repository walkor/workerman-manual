# 如何广播数据

## 范例（定时广播）

```php
use Workerman\Worker;
use Workerman\Lib\Timer;
require_once './Workerman/Autoloader.php';

$worker = new Worker('text://0.0.0.0:2020');
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

## 范例（群聊）

```php
use Workerman\Worker;
use Workerman\Lib\Timer;
require_once './Workerman/Autoloader.php';

$worker = new Worker('text://0.0.0.0:2020');
// 客户端发来消息时，广播给其它用户
$worker->onMessage = function($connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// 运行worker
Worker::runAll();
```
