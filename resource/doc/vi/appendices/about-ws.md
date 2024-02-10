# Giao thức ws

Hiện tại, phiên bản giao thức **ws của Workerman là 13**.

Workerman có thể được sử dụng như một máy khách, thông qua giao thức ws để thiết lập kết nối websocket với máy chủ websocket từ xa, để thực hiện giao tiếp hai chiều.

> **Chú ý**
> Giao thức ws chỉ có thể được sử dụng như là một máy khách thông qua AsyncTcpConnection, không thể sử dụng làm giao thức lắng nghe máy chủ websocket. Nghĩa là cách viết sau là không đúng.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Nếu muốn sử dụng Workerman như là máy chủ websocket, vui lòng sử dụng [giao thức websocket](about-websocket.md).

**Ví dụ về việc sử dụng giao thức ws làm máy khách websocket:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Khi quá trình khởi động worker
$worker->onWorkerStart = function()
{
    // Thiết lập kết nối websocket với máy chủ websocket từ xa
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Gửi một heartbeat websocket với opcode là 0x9 đến máy chủ sau mỗi 55 giây (tùy chọn)
    $ws_connection->websocketPingInterval = 55;
    // Thiết lập header HTTP (tùy chọn)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Thiết lập loại dữ liệu (tùy chọn)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB cho dạng text BINARY_TYPE_ARRAYBUFFER cho dạng nhị phân
    // Khi kết nối TCP hoàn thành ba bước bắt tay (tùy chọn)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // Khi hoàn tất bắt tay websocket (tùy chọn)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Khi nhận được tin nhắn từ máy chủ websocket từ xa
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // Khi có lỗi xảy ra trong quá trình kết nối, thường là lỗi kết nối máy chủ websocket từ xa thất bại (tùy chọn)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Khi kết nối với máy chủ websocket từ xa bị đóng (tùy chọn, nên thêm để kết nối lại)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // Nếu kết nối bị đóng, thử kết nối lại sau 1 giây
        $connection->reConnect(1);
    };
    // Sau khi thiết lập các gọi lại trên, thực hiện việc kết nối
    $ws_connection->connect();
};
Worker::runAll();
```

Xem thêm tại [Làm máy khách ws/wss](../faq/as-wss-client.md)
