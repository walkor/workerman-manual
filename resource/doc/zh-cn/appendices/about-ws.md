# ws协议

目前Workerman的**ws协议版本为13**。

workerman可以作为客户端，通过ws协议发起websocket连接，连到远程websocket服务器，实现双向通讯。

> **注意**
> ws协议只能通过AsyncTcpConnection作为客户端使用，不能作为websocket服务端监听协议。也就是说以下写法是错误的。 

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

如果想用workerman作为websocket服务端，请使用[websocket协议](about-websocket.md)。

**ws作为websocket客户端协议示例：**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// 进程启动时
$worker->onWorkerStart = function()
{
    // 以websocket协议连接远程websocket服务器
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // 每隔55秒向服务端发送一个opcode为0x9的websocket心跳(可选)
    $ws_connection->websocketPingInterval = 55;
    // 设置http头(可选)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // 设置数据类型(可选)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB为文本 BINARY_TYPE_ARRAYBUFFER为二进制
    // 当TCP完成三次握手后(可选)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // 当websocket完成握手后(可选)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // 远程websocket服务器发来消息时
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // 连接上发生错误时，一般是连接远程websocket服务器失败错误(可选)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // 当连接远程websocket服务器的连接断开时(可选，建议加上重连)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // 如果连接断开，1秒后重连
        $connection->reConnect(1);
    };
    // 设置好以上各种回调后，执行连接操作
    $ws_connection->connect();
};
Worker::runAll();
```

更多参考[作为ws/wss客户端](../faq/as-wss-client.md)

