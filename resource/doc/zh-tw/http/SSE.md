# SSE 
**此功能需要workerman>=4.0.0**

SSE也就是Server-sent Events，是一種伺服器推送技術。它的本質是客戶端發送一個攜帶`Accept: text/event-stream`頭的http請求後，連接不關閉，伺服器可以在這個連接上不斷的給客戶端推送數據。

它與websocket的區別是：
*   SSE只能伺服器向客戶端推；WebSocket可以雙向通訊。
*   SSE 默認支持斷線重連；WebSocket 需要自己實現。
*   SSE 只能傳輸utf8文本，二進制數據需要編碼成utf8後傳送；WebSocket 默認支持傳送utf8和二進制數據。
*   SSE 自帶消息類型；WebSocket 需要自己實現。

### 例子
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
    // 如果Accept頭是text/event-stream則說明是SSE請求
    if ($request->header('accept') === 'text/event-stream') {
        // 首先發送一個 Content-Type: text/event-stream 頭的響應
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // 定時向客戶端推送數據
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // 連接關閉的時候要將定時器刪除，避免定時器不斷累積導致內存洩漏
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // 發送message事件，事件攜帶的數據為hello，消息id可以不傳
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// 運行worker
Worker::runAll();
```

客戶端javascript代碼
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // 輸出 hello
}, false);
```
