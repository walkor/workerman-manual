# SSE 
**Tính năng này yêu cầu workerman>=4.0.0**

SSE hay Server-sent Events, là một công nghệ push từ máy chủ. Bản chất của nó là khi máy khách gửi một yêu cầu HTTP có tiêu đề `Accept: text/event-stream`, kết nối không đóng, máy chủ có thể liên tục đẩy dữ liệu đến máy khách thông qua kết nối này.

SSE khác biệt với websocket như sau:
*   SSE chỉ cho phép máy chủ đẩy dữ liệu đến máy khách; Websocket có thể trò chuyện hai chiều.
*   SSE mặc định hỗ trợ việc kết nối lại khi mất kết nối; Websocket cần phải tự triển khai.
*   SSE chỉ có thể truyền dữ liệu văn bản utf8, dữ liệu nhị phân cần phải được mã hóa thành utf8 để truyền; Websocket mặc định hỗ trợ truyền dữ liệu utf8 và nhị phân.
*   SSE có loại tin nhắn tích hợp; Websocket cần phải tự triển khai.

### Ví dụ
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
    // Nếu tiêu đề Accept là text/event-stream thì đây là yêu cầu SSE
    if ($request->header('accept') === 'text/event-stream') {
        // Trước tiên gửi một phản hồi có tiêu đề Content-Type: text/event-stream
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Định kỳ đẩy dữ liệu đến máy khách
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // Khi kết nối đóng thì cần xóa định kỳ để tránh tích lũy và gây rò rỉ bộ nhớ
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Gửi sự kiện message với dữ liệu là hello, id tin nhắn có thể không cần thiết
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Chạy worker
Worker::runAll();
```

Mã JavaScript trên máy khách
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // Xuất ra hello
}, false);
```
