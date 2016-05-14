# ws协议

workerman可以作为客户端通过ws协议发起websocket连接，连到远程websocket服务器，实现双向通讯。

注意：ws协议只能通过AsyncTcpConnection作为客户端使用，不能作为websocket服务端监听协议。也就是说以下写法是错误的。
```php
$worker = new Worker('ws://0.0.0.0:8080');
```

如果想workerman作为websocket服务端，请使用[websocket协议](/appendices/about-websocket.html)。

**ws作为websocket客户端协议示例：**
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/Workerman/Autoloader.php';
$worker = new Worker();
// 进程启动时
$worker->onWorkerStart = function()
{
    // 以websocket协议连接远程websocket服务器
    $ws_connection = new AsyncTcpConnection("ws://echo.websocket.org:80");
    // 连上后发送hello字符串
    $ws_connection->onConnect = function($connection){
        $connection->send('hello');
    };
    // 远程websocket服务器发来消息时
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // 连接上发生错误时，一般是连接远程websocket服务器失败错误
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // 当连接远程websocket服务器的连接断开时
    $ws_connection->onClose = function($connection){
        echo "connection closed\n";
    };
    // 设置好以上各种回调后，执行连接操作
    $ws_connection->connect();
};
Worker::runAll();
```

