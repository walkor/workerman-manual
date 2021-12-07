# 心跳

注意：长连接应用必须加心跳，否则连接可能由于长时间未通讯被路由节点强行断开。

心跳作用主要有两个：

1、客户端定时给服务端发送点数据，防止连接由于长时间没有通讯而被某些节点的防火墙关闭导致连接断开的情况。

2、服务端可以通过心跳来判断客户端是否在线，如果客户端在规定时间内没有发来任何数据，就认为客户端下线。这样可以检测到客户端由于极端情况(断电、断网等)下线的事件。

心跳间隔建议值：

建议客户端发送心跳间隔小于60秒，比如55秒。

> 心跳的数据格式没有要求，服务端能识别即可。

## 心跳示例
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 心跳间隔55秒
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // 给connection临时设置一个lastMessageTime属性，用来记录上次收到消息的时间
    $connection->lastMessageTime = time();
    // 其它业务逻辑...
};

// 进程启动后设置一个每10秒运行一次的定时器
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // 有可能该connection还没收到过消息，则lastMessageTime设置为当前时间
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // 上次通讯时间间隔大于心跳间隔，则认为客户端已经下线，关闭连接
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

以上配置为如果客户端超过55秒没有发送任何数据给服务端，则服务端认为客户端已经掉线，服务端关闭连接并触发onClose。
