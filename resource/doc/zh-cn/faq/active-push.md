# 如何主动推送消息

1、可以用定时器定时推送数据
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// 进程启动后定时推送数据给客户端
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2、其它项目中发生某个事件时通知workerman推送数据
参见 [常见问题-在其它项目中推送](push-in-other-project.md)
