# onMessage
```php
callback \Channel\Client::$onMessage
```
当有订阅消息到来时的回调

## 回调函数的参数

``` $subject ```

是哪个主题的消息。类型为字符串

``` $data ```

消息内容。可能是字符串或者数组


## 范例
多进程Worker（多服务器），一个客户端发消息，广播给所有客户端


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
$worker->name = 'websocket';
$worker->count = 6;
$worker->onWorkerStart = function($worker)
{
    // Channel客户端连接到Channel服务端
    Channel\Client::connect('127.0.0.1', 2206);
    // 订阅主题为broadcast的消息
    Channel\Client::subscribe('broadcast');
    // 当all_message主题有消息时触发的回调
    Channel\Client::$onMessage = function($subject, $message)use($worker){
        // 向所有客户端广播消息
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    };
};

$worker->onMessage = function($connection, $data)
{
   \Channel\Client::publish('broadcast', $data);
};

Worker::runAll();
```

**测试**

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)

接收消息的连接
```javascript
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

广播消息
```
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onopen = function()
{
    ws.send('hello world');
};
```



