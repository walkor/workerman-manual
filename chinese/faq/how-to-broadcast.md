# 如何广播(群发)数据

## 范例（定时广播）

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// 这个例子中进程数必须为1
$worker->count = 1;
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
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// 这个例子中进程数必须为1
$worker->count = 1;
// 客户端发来消息时，广播给其它用户
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// 运行worker
Worker::runAll();
```

## 说明：
**单进程：**
以上例子只能**单进程**（```$worker->count=1```），因为多进程时多个客户端可能连接到不同的进程中，进程间的客户端是隔离的，无法直接通讯，也就是A进程中无法**直接**操作B进程的客户端connection对象发送数据。(要做到这点，需要进程间通讯，比如可以使用Channel组件，例如[例子-集群发送](components/channel-examples.md)、[例子-分组发送](components/channel-examples2.md))。

**建议用GatewayWorker**
在workerman基础上开发的GatewayWoker框架提供了更方便推送机制，包括组播、广播等，可以设置多进程甚至可以多服务器部署，如果需要给客户端推送数据，建议使用GatewayWorker框架。

GatewayWorker手册地址 https://doc2.workerman.net 

