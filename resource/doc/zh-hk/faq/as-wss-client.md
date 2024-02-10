# 作為ws/wss客戶端

有時候需要讓workerman作為客戶端以ws/wss協議去連接某個服務端，並與之交互。
以下是示例。

## workerman作為ws客戶端

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // websocket握手成功後
    $con->onWebSocketConnect = function(AsyncTcpConnection $con, ) {
        $con->send('hello');
    };

    // 當收到消息時
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```


## workerman作為wss(ws+ssl)客戶端

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ssl需要訪問443端口
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // 設置以ssl加密方式訪問，使之成為wss
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


## workerman作為wss(ws+ssl)客戶端+本地ssl證書

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 設置訪問對方主機的本地ip及端口以及ssl證書
    $context_option = array(
        // ssl選項，參考http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // 本地證書路徑。 必須是 PEM 格式，並且包含本地的證書及私鑰。
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert 文件的密碼。
            'passphrase'        => 'your_pem_passphrase',
            // 是否允許自簽名證書。
            'allow_self_signed' => true,
            // 是否需要驗證 SSL 證書。
            'verify_peer'       => false
        )
    );

    // ssl需要訪問443端口
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // 設置以ssl加密方式訪問，使之成為wss
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


## 其它設置
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// 進程啟動時
$worker->onWorkerStart = function()
{
    // 以websocket協議連接遠程websocket服務器
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // 每隔55秒向服務端發送一個opcode為0x9的websocket心跳
    $ws_connection->websocketPingInterval = 55;
    // 自定義http頭
    $ws_connection->headers = ['token' => 'value'];
    // 設置數據類型，默認BINARY_TYPE_BLOB為文本
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB為文本 BINARY_TYPE_ARRAYBUFFER為二進制
    // 當TCP完成三次握手後
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // 當websocket完成握手後
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // 遠程websocket服務器發來消息時
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // 連接上發生錯誤時，一般是連接遠程websocket服務器失敗錯誤
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // 當連接遠程websocket服務器的連接斷開時
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // 如果連接斷開，1秒後重連
        $connection->reConnect(1);
    };
    // 設置好以上各種回調後，執行連接操作
    $ws_connection->connect();
};
Worker::runAll();
```
