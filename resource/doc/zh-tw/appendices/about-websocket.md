# WebSocket協議

目前Workerman的**WebSocke協議版本為13**。

WebSocket protocol 是HTML5一種新的協議。它實現了瀏覽器與伺服器全雙工通信。

## WebSocket與TCP關係

WebSocket和HTTP一樣是一種應用層協議，都是基於TCP傳輸的，WebSocket本身和Socket並沒有多大關係，更不能等同。

## WebSocket協議握手

WebSocket協議有一個握手的過程，握手時瀏覽器和服務端是以HTTP協議通信的，在Workerman中可以這樣介入到握手過程。

**當 workerman <= 4.1 時**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // 可以在這裡判斷連接來源是否合法，不合法就關掉連接
        // $_SERVER['HTTP_ORIGIN']標識來自哪個站點的頁面發起的websocket連接
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // onWebSocketConnect 裡面$_GET $_SERVER是可用的
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**當 workerman >= 5.0 時**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## WebSocket協議傳輸二進位數據

websocket協議默認只能傳輸utf8文本，如果要傳輸二進位數據，請閱讀以下部分。

websocket協議中在協議頭中使用一個標記位來標記傳輸的是二進位數據還是utf8文本數據，瀏覽器會驗證標記和傳輸的內容類型是否符合，如果不符合則會報錯斷開連接。

所以服務端發送數據的時候需要根據傳輸的數據類型設置這個標記位，在Workerman中如果是普通utf8文本，則需要設置（默認就是此值，一般不用再手動設置）
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

如果是二進位數據，則需要設置
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**注意**：如果沒設置$connection->websocketType，則$connection->websocketType默認為BINARY_TYPE_BLOB（也就是utf8文本類型）。一般應用傳輸的都是utf8文本，例如傳輸的是json數據，所以不用手動設置$connection->websocketType。只有在傳輸二進位數據時（例如圖片數據、protobuffer數據等）才要設置此屬性為BINARY_TYPE_ARRAYBUFFER。

## 把workerman作為Websocket客戶端

可以利用[AsyncTcpConnection類](../async-tcp-connection.md)配合[ws協議](about-ws.md)讓workerman作為websocket客戶端連接遠程websocket服務端，完成雙向實時通訊。
