# WebSocketクライアント

Workermanは、時にはクライアントとしてWebSocket（ws/wss）プロトコルを使用してサーバーに接続し、対話を行う必要があります。以下に例を示します。

## Workermanがwsクライアントとして動作する場合

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // WebSocketハンドシェイク成功後
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // メッセージを受信した場合
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workermanがwss（ws+ssl）クライアントとして動作する場合

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // SSLは443ポートにアクセスする必要があります
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // SSLでアクセスするように設定し、wssになるようにします
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

## Workermanがwss（ws+ssl）クライアントかつローカルSSL証明書を使用する場合
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $context_option = array(
        'ssl' => array(
            'local_cert'        => '/your/path/to/pemfile',
            'passphrase'        => 'your_pem_passphrase',
            'allow_self_signed' => true,
            'verify_peer'       => false
        )
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

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

## その他の設定
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();

$worker->onWorkerStart = function()
{
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    $ws_connection->websocketPingInterval = 55;
    $ws_connection->headers = ['token' => 'value'];
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB;
    $ws_connection->onConnect = function($connection){
        echo "TCP connected\n";
    };
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    $ws_connection->onMessage = function($connection, $data){
        echo "Recv: $data\n";
    };
    $ws_connection->onError = function($connection, $code, $msg){
        echo "Error: $msg\n";
    };
    $ws_connection->onClose = function($connection){
        echo "Connection closed and try to reconnect\n";
        $connection->reConnect(1);
    };
    $ws_connection->connect();
};

Worker::runAll();
```
