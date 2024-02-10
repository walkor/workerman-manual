# WebSocketプロトコル

現在、WorkermanのWebSocketプロトコルのバージョンは13です。

WebSocketプロトコルは、HTML5の新しいプロトコルです。これにより、ブラウザとサーバー間での双方向通信が実現されます。

## WebSocketとTCPの関係

WebSocketはHTTP同様にアプリケーション層プロトコルであり、TCPを基にしています。WebSocket自体はSocketとは直接関係がなく、等しいとは言えません。

## WebSocketプロトコルのハンドシェイク

WebSocketプロトコルにはハンドシェイクのプロセスがあり、ハンドシェイク中にはブラウザとサーバーがHTTPプロトコルで通信します。Workermanでは、ハンドシェイクプロセスに以下のように介入できます。

**Workerman 4.1以下の場合**

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
        // ここで接続元の正当性を判断し、不正な場合は接続を閉じることができます。
        // $_SERVER['HTTP_ORIGIN'] はWebSocket接続を開始したサイトを示します。
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // onWebSocketConnect の中で $_GET $_SERVER が利用可能です
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Workerman 5.0以上の場合**

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

## WebSocketプロトコルでバイナリデータを転送する

WebSocketプロトコルでは、デフォルトでutf8テキストのみを転送することができますが、バイナリデータを転送する場合は以下を参照してください。

WebSocketプロトコルでは、プロトコルヘッダーで転送するデータがバイナリデータかUTF-8テキストデータかを示すフラグを使用し、ブラウザはこのフラグと転送される内容の型を検証し、一致しない場合は接続を切断します。

そのため、サーバーがデータを送信する際には、転送するデータのタイプに応じてこのフラグを設定する必要があります。Workermanでは、テキストデータの場合は設定する必要はありません（デフォルトの値ですが、通常は手動で設定する必要はありません）。

```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

バイナリデータの場合は、以下を設定する必要があります。

```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**注意**: もし$connection->websocketTypeを設定しない場合、$connection->websocketTypeはデフォルトでBINARY_TYPE_BLOB（すなわちUTF-8テキスト型）になります。通常、アプリケーションで転送されるデータはUTF-8テキストであり、例えばJSONデータを転送する場合、$connection->websocketTypeを手動で設定する必要はありません。バイナリデータ（例えば画像データ、プロトバッファーデータなど）を転送する場合にのみ、このプロパティをBINARY_TYPE_ARRAYBUFFERに設定する必要があります。

## WorkermanをWebSocketクライアントとして使用する
[AsyncTcpConnectionクラス](../async-tcp-connection.md)と[wsプロトコル](about-ws.md)を利用して、WorkermanをWebSocketクライアントとしてリモートWebSocketサーバーに接続し、双方向のリアルタイム通信を実現することができます。
