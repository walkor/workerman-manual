# on
**``` (要求Workerman版本>=3.3.0) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
订阅```$event_name```事件并注册事件发生时的回调```$callback_function```

## 回调函数的参数

``` $event_name ```

订阅的事件名称，可以是任意的字符串。

``` $callback_function ```

事件发生时触发的回调函数。函数原型为```callback_function(mixed $event_data)```。```$event_data```是事件发布(publish)时传递的事件数据。


注意：

如果同一个事件注册了两个回调函数，后一个回调函数将覆盖前一个回调函数。


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
// 每个worker进程启动时
$worker->onWorkerStart = function($worker)
{
    // Channel客户端连接到Channel服务端
    Channel\Client::connect('127.0.0.1', 2206);
    // 订阅broadcast事件，并注册事件回调
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // 向当前worker进程的所有客户端广播消息
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function($connection, $data)
{
   // 将客户端发来的数据当做事件数据
   $event_data = $data;
   // 向所有worker进程发布broadcast事件
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**测试**

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)

接收消息的连接
```javascript
// 127.0.0.1换成实际workerman所在ip
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```

广播消息
```
ws.send('hello world');
```



