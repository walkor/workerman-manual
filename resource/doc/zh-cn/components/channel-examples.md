# 例子1
**``` (要求Workerman版本>=3.3.0) ```**

基于Worker的多进程(分布式集群)推送系统，集群群发、集群广播。

`start_channel.php`
整个系统只能部署一个start_channel服务。假设运行在192.168.1.1。
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// 初始化一个Channel服务端
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
整个系统可以部署多个start_ws服务，假设运行在 192.168.1.2和192.168.1.3 两台服务器上。
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket服务端
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Channel客户端连接到Channel服务端
    Channel\Client::connect('192.168.1.1', 2206);
    // 以自己的进程id为事件名称
    $event_name = $worker->id;
    // 订阅worker->id事件并注册事件处理函数
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "connection not exists\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // 订阅广播事件
    $event_name = '广播';
    // 收到广播事件后向当前进程内所有客户端连接发送广播数据
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} connected\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
整个系统可以部署多个start_http服务，假设运行在 192.168.1.4和192.168.1.5 两台服务器上。
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 用来处理http请求，向任意客户端推送数据，需要传workerID和connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // 兼容workerman4.x 5.x
    if (!is_array($request)) {
        $get = $request->get();
    }
    $connection->send('ok');
    if(empty($get['content'])) return;
    // 是向某个worker进程中某个连接推送数据
    if(isset($get['to_worker_id']) && isset($get['to_connection_id']))
    {
        $event_name = $get['to_worker_id'];
        $to_connection_id = $get['to_connection_id'];
        $content = $get['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // 是全局广播数据
    else
    {
        $event_name = '广播';
        $content = $get['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## 测试 
1、运行个个服务器上的服务

2、客户端连接服务端

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)

```javascript
// 也可以连ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

3、通过调用http接口推送

url访问 ```http://192.168.1.4:4237/?content={$content}```  或者  ```http://192.168.1.5:4237/?content={$content}```向所有客户端连接推送```$content```数据

url访问 ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` 或者```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` 向某个worker进程中的某个客户端连接推送```$content```数据

注意：测试时把 ```{$worker_id}``` ```{$connection_id}``` 和```{$content}``` 换成实际值

