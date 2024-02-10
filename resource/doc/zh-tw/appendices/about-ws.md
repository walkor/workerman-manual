# ws協議

目前Workerman的**ws協議版本為13**。

Workerman可以作為客戶端，透過ws協議發起websocket連接，連到遠程websocket伺服器，實現雙向通訊。

> **注意**
> ws協議只能透過AsyncTcpConnection作為客戶端使用，不能作為websocket服務端監聽協議。也就是說以下寫法是錯誤的。

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

如果想用Workerman作為websocket服務端，請使用[websocket協議](about-websocket.md)。

**ws作為websocket客戶端協議示例：**

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
    // 以websocket協議連接遠程websocket伺服器
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // 每隔55秒向服務端發送一個opcode為0x9的websocket心跳(可選)
    $ws_connection->websocketPingInterval = 55;
    // 設置http頭(可選)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // 設置資料類型(可選)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB為文本 BINARY_TYPE_ARRAYBUFFER為二進制
    // 當TCP完成三次握手後(可選)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // 當websocket完成握手後(可選)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // 遠程websocket伺服器發來消息時
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // 連接上發生錯誤時，一般是連接遠程websocket伺服器失敗錯誤(可選)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // 當連接遠程websocket伺服器的連接斷開時(可選，建議加上重連)
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

更多參考[作為ws/wss客戶端](../faq/as-wss-client.md)
