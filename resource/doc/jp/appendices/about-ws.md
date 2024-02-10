# wsプロトコル

現在のWorkermanの**wsプロトコルのバージョンは13**です。

Workermanはクライアントとして機能し、wsプロトコルを使用してWebSocket接続を開始し、リモートWebSocketサーバーに接続して双方向通信を実現することができます。

> **注意**
> wsプロトコルはAsyncTcpConnection経由でのみクライアントとして使用でき、WebSocketサーバーでのリッスンプロトコルとしては使用できません。つまり、以下のような書き方は間違っています。

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

WorkermanをWebSocketサーバーとして使用したい場合は、[websocketプロトコル](about-websocket.md)を使用してください。

**wsをWebSocketクライアントプロトコルとして使用する例：**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// ワーカー起動時
$worker->onWorkerStart = function()
{
    // WebSocketプロトコルを使用してリモートWebSocketサーバーに接続
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // 55秒ごとにサーバーにopcodeが0x9のWebSocketハートビートを送信（オプション）
    $ws_connection->websocketPingInterval = 55;
    // HTTPヘッダーを設定（オプション）
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // データタイプを設定（オプション）
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOBはテキスト、BINARY_TYPE_ARRAYBUFFERはバイナリ
    // TCPの三方握手が完了した後（オプション）
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // WebSocketのハンドシェイクが完了した後（オプション）
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // リモートWebSocketサーバーからメッセージを受信した時
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // 接続エラーが発生した際に実行される。一般的には、リモートWebSocketサーバーへの接続失敗エラー（オプション）
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // リモートWebSocketサーバーへの接続が切断された場合に実行される（オプション、再接続をお勧めします）
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // 接続が切断された場合、1秒後に再接続
        $connection->reConnect(1);
    };
    // 上記のコールバックを設定した後、接続処理を実行
    $ws_connection->connect();
};
Worker::runAll();
```

詳細は[ws/wssクライアントとして](../faq/as-wss-client.md)を参照してください。
