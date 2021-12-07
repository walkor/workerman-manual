# SSE 
**此特性需要workerman>=4.0.0**

SSE也就是Server-sent Events，是一种服务端推送技术。它的本质是客户端发送一个携带`Accept: text/event-stream` 头的http请求后，连接不关闭，服务端可以在这个连接上不断的给客户端推送数据。

它与websocket的区别是：
*   SSE只能服务端向客户端推；Websocket可以双向通讯。
*   SSE 默认支持断线重连；WebSocket 需要自己实现。
*   SSE 只能传输utf8文本，二进制数据需要编码成utf8后传送；WebSocket 默认支持传送utf8和二进制数据。
*   SSE 自带消息类型；WebSocket 需要自己实现。

### 例子
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // 如果Accept头是text/event-stream则说明是SSE请求
    if ($request->header('accept') === 'text/event-stream') {
        // 首先发送一个 Content-Type: text/event-stream 头的响应
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream']));
        // 定时向客户端推送数据
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // 连接关闭的时候要将定时器删除，避免定时器不断累积导致内存泄漏
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // 发送message事件，事件携带的数据为hello，消息id可以不传
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// 运行worker
Worker::runAll();
```

客户端javascript代码
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // 输出 hello
}, false);
```


