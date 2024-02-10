# SSE 
**この機能にはworkerman>=4.0.0が必要です**

SSE（Server-sent Events）は、サーバーからのプッシュ技術です。その本質は、クライアントが`Accept: text/event-stream` ヘッダーを含むHTTPリクエストを送信すると、接続が閉じられず、サーバーはこの接続でクライアントにデータを継続的にプッシュできるというものです。

WebSocketとの違いは次のとおりです：
*   SSEはサーバーからクライアントにのみプッシュできます；WebSocketは双方向通信が可能です。
*   SSEはデフォルトで再接続をサポートしています；WebSocketは再接続を自分で実装する必要があります。
*   SSEはUTF-8テキストのみを転送できます。バイナリデータはUTF-8にエンコードして送信する必要があります；WebSocketはデフォルトでUTF-8とバイナリデータの転送をサポートしています。
*   SSEにはメッセージタイプが含まれています；WebSocketは自分で実装する必要があります。

### 例
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // もしAcceptヘッダーがtext/event-streamであれば、SSEリクエストであると判断
    if ($request->header('accept') === 'text/event-stream') {
        // まずContent-Type: text/event-streamヘッダーを送信
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // 定期的にクライアントにデータをプッシュ
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // 接続が閉じられた場合は、タイマーを削除してメモリリークを防ぐ
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // messageイベントを送信し、データにはhelloを含む。メッセージIDは省略可能
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// ワーカーを実行
Worker::runAll();
```

クライアント側のJavaScriptコード
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // 出力：hello
}, false);
```
