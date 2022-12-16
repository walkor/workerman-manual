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

## 断线重连(重要)

不管是客户端发送心跳还是服务端发送心跳，连接都有断开的可能。例如浏览器最小化js被暂停、浏览器切换到其它tab页面js被暂停、电脑进入睡眠等等、移动端切换网络、信号变弱、手机黑屏、手机应用切换到后台、路由故障、业务主动断开等。尤其是外网环境复杂，很多路由节点会清理1分钟内不活跃的连接，这也是为什么心跳间隔推荐小于1分钟的原因。

连接在外网环境很容易被断开，所以断线重连是长连接应用必须具备的功能(断线重连只能客户端做，服务端无法实现)。例如浏览器websocket需要监听onclose事件，当发生onclose时建立新的连接(为避免需崩可延建立连接)。更严格一点，服务端也应该定时发起心跳数据，并且客户端需要定时监测服务端的心跳数据是否超时，超过规定时间未收到服务端心跳数据应该认定连接已经断开，需要执行close关闭连接，并重新建立新的连接。
