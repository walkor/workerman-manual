# 作为ws/wss客户端

有时候需要让workerman作为客户端以ws/wss协议去连接某个服务端，并与之交互。
以下是示例。



## workerman作为ws客户端

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // websocket握手成功后
    $con->onWebSocketConnect = function(AsyncTcpConnection $con, ) {
        $con->send('hello');
    };

    // 当收到消息时
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## workerman作为wss(ws+ssl)客户端

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ssl需要访问443端口
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // 设置以ssl加密方式访问，使之成为wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```


## workerman作为wss(ws+ssl)客户端+本地ssl证书

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 设置访问对方主机的本地ip及端口以及ssl证书
    $context_option = array(
        // ssl选项，参考http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // 本地证书路径。 必须是 PEM 格式，并且包含本地的证书及私钥。
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert 文件的密码。
            'passphrase'        => 'your_pem_passphrase',
            // 是否允许自签名证书。
            'allow_self_signed' => true,
            // 是否需要验证 SSL 证书。
            'verify_peer'       => false
        )
    );

    // ssl需要访问443端口
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // 设置以ssl加密方式访问，使之成为wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## 其它设置
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
    // 每隔55秒向服务端发送一个opcode为0x9的websocket心跳
    $ws_connection->websocketPingInterval = 55;
    // 自定义http头
    $ws_connection->headers = ['token' => 'value'];
    // 设置数据类型，默认BINARY_TYPE_BLOB为文本
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB为文本 BINARY_TYPE_ARRAYBUFFER为二进制
    // 当TCP完成三次握手后
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // 当websocket完成握手后
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
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
        echo "connection closed and try to reconnect\n";
        // 如果连接断开，1秒后重连
        $connection->reConnect(1);
    };
    // 设置好以上各种回调后，执行连接操作
    $ws_connection->connect();
};
Worker::runAll();
```


