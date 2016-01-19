# 例子1
基于Worker的多进程(分布式集群)推送系统

start.php

```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';
require_once './Channel/src/Server.php';
require_once './Channel/src/Client.php';

// 初始化一个Channel服务端
$channel_server = new Channel\Server('0.0.0.0', 2206);

// websocket服务端
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Channel客户端连接到Channel服务端
    Channel\Client::connect('127.0.0.1', 2206);
    // 订阅当前进程workerID的消息
    Channel\Client::subscribe($worker->id);
    // 当前workerID有消息时触发的回调
    Channel\Client::$onMessage = function($channel, $data)use($worker){
        $to_connection_id = $data['to_connection_id'];
        $message = $data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "connection not exsits\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    };
};

$worker->onConnect = function($connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} connected\n";
    echo $msg;
    $connection->send($msg);
};


// 用来处理http请求，向任意客户端推送数据，需要传workerID和connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function($connection, $data)
{
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    $to_worker_id = $_GET['to_worker_id'];
    $to_connection_id = $_GET['to_connection_id'];
    $content = $_GET['content'];
    Channel\Client::publish($to_worker_id, array(
       'to_connection_id' => $to_connection_id,
       'content'          => $content
    ));
};

Worker::runAll();
```

## 测试 （假设都是本机127.0.0.1运行）
1、运行服务单
```
 php start.php start
Workerman[start.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
root          ChannelServer  text://0.0.0.0:2206        1         [OK]
root          pusher         websocket://0.0.0.0:4236   2         [OK]
root          publisher      http://0.0.0.0:4237        1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

2、客户端连接服务端

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)

```javascript
// 假设服务端ip为127.0.0.1
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

3、通过调用http接口推送

浏览器访问 http://127.0.0.1:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content} 完成推送

注意：把```{$worker_id}``` ```{$connection_id}``` 和```{$content}``` 换成实际值

