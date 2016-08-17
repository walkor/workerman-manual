# listen
```php
void Worker::listen(void)
```
用于实例化Worker后执行监听。

此方法主要用于在Worker进程启动后动态创建新的Worker实例，能够实现同一个进程监听多个端口，支持多种协议。

例如一个http Worker启动后实例化一个websocket Worker，那么这个进程即能通过http协议访问，又能通过websocket协议访问。由于websocket Worker和http Worker在同一个进程中，所以它们可以访问共同的内存变量，共享所有socket连接。可以做到接收http请求，然后操作websocket客户端完成向客户端推送数据类似的效果。

**注意：**

如果PHP版本<=7.0，则不支持在多个子进程中实例化相同端口的Worker。例如A进程创建了监听2016端口的Worker，那么B进程就不能再创建监听2016端口的Worker，否则会报```Address already in use```错误。例如下面的代码是无法运行的。

```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4个进程
$worker->count = 4;
// 每个进程启动后在当前进程新增一个Worker监听
$worker->onWorkerStart = function($worker)
{
    /**
     * 4个进程启动的时候都创建2016端口的Worker
     * 当执行到worker->listen()时会报Address already in use错误
     * 如果worker->count=1则不会报错
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // 执行监听。这里会报Address already in use错误
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message($connection, $data)
{
    $connection->send("hello\n");
}

// 运行worker
Worker::runAll();
```

如果您的PHP版本>=7.0，可以设置Worker->reusePort=true， 这样可以做到多个子进程创建相同端口的Worker。见下面的例子：
```php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4个进程
$worker->count = 4;
// 每个进程启动后在当前进程新增一个Worker监听
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // 设置端口复用，可以创建监听相同端口的Worker（需要PHP>=7.0）
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // 执行监听。正常监听不会报错
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message($connection, $data)
{
    $connection->send("hello\n");
}

// 运行worker
Worker::runAll();
```


### 示例 php后端及时推送消息给客户端

**原理：**

1、建立一个websocket Worker，用来维持客户端长连接

2、websocket Worker内部建立一个text Worker

3、websocket Worker 与 text Worker是同一个进程，可以方便的共享客户端连接

4、某个独立的php后台系统通过text协议与text Worker通讯

5、text Worker操作websocket连接完成数据推送

**代码及步骤**

push.php

```php
<?php
use Workerman\Worker;
require_once './Workerman/Autoloader.php';
// 初始化一个worker容器，监听1234端口
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * 注意这里进程数必须设置为1，否则会报端口占用错误
 * (php 7可以设置进程数大于1，前提是$inner_text_worker->reusePort=true)
 */
$worker->count = 1;
// worker进程启动后创建一个text Worker以便打开一个内部通讯端口
$worker->onWorkerStart = function($worker)
{
    // 开启一个内部端口，方便内部系统推送数据，Text协议格式 文本+换行符
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function($connection, $buffer)
    {
        // $data数组格式，里面有uid，表示向那个uid的页面推送数据
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // 通过workerman，向uid的页面推送数据
        $ret = sendMessageByUid($uid, $buffer);
        // 返回推送结果
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## 执行监听 ##
    $inner_text_worker->listen();
};
// 新增加一个属性，用来保存uid到connection的映射
$worker->uidConnections = array();
// 当有客户端发来消息时执行的回调函数
$worker->onMessage = function($connection, $data)
{
    global $worker;
    // 判断当前客户端是否已经验证,既是否设置了uid
    if(!isset($connection->uid))
    {
       // 没验证的话把第一个包当做uid（这里为了方便演示，没做真正的验证）
       $connection->uid = $data;
       /* 保存uid到connection的映射，这样可以方便的通过uid查找connection，
        * 实现针对特定uid推送数据
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// 当有客户端连接断开时
$worker->onClose = function($connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // 连接断开时删除映射
        unset($worker->uidConnections[$connection->uid]);
    }
};

// 向所有验证的用户推送数据
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// 针对uid推送数据
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// 运行所有的worker
Worker::runAll();
```

启动后端服务
```php push.php start -d```

前端接收推送的js代码
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

后端推送消息的代码
```php
// 建立socket连接到内部推送端口
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// 推送的数据，包含uid字段，表示是给这个uid推送
$data = array('uid'=>'uid1', 'percent'=>'88%');
// 发送数据，注意5678端口是Text协议的端口，Text协议需要在数据末尾加上换行符
fwrite($client, json_encode($data)."\n");
// 读取推送结果
echo fread($client, 8192);
```
