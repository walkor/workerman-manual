# 心跳
心跳作用主要有两个：

1、客户端定时给服务端发送点数据，防止连接由于长时间没有通讯而被某些节点的防火墙关闭导致连接断开的情况。

2、服务端可以通过心跳来判断客户端是否在线，如果客户端在规定时间内没有发来任何数据，就认为客户端下线。这样可以检测到客户端由于极端情况(断电、断网等)下线的事件。



## 心跳示例
```php
<?php
require_once '/your/path/Workerman/Autoloader.php';
use Workerman\Worker;
use Workerman\Lib\Timer;

// 心跳间隔25秒
define('HEARTBEAT_TIME', 25);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function($connection, $msg) {
    // 给connection临时设置一个lastMessageTime属性，用来记录上次收到消息的时间
    $connection->lastMessageTime = time();
    // 其它业务逻辑...
};

// 进程启动后设置一个每秒运行一次的定时器
$worker->onWorkerStart = function($worker) {
    Timer::add(1, function()use($worker){
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
