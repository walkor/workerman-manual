# Giao thức WebSocket

Hiện tại, phiên bản giao thức WebSocket của **Workerman là 13**.

Giao thức WebSocket là một giao thức mới trong HTML5. Nó thực hiện việc giao tiếp hai chiều giữa trình duyệt và máy chủ.

## Mối quan hệ giữa WebSocket và TCP

WebSocket, giống như HTTP, là một giao thức ứng dụng, cả hai đều dựa trên việc truyền qua TCP, WebSocket chính nó không có mối quan hệ lớn với Socket, và không thể tương đương.

## Bắt tay vào giao thức WebSocket

Giao thức WebSocket có một quá trình bắt tay, khi bắt tay, trình duyệt và máy chủ giao tiếp theo giao thức HTTP. Trong Workerman, bạn có thể can thiệp vào quá trình bắt tay như sau.

**Khi workerman <= 4.1**
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
        // Có thể kiểm tra xem nguồn kết nối có hợp lệ hay không, nếu không hợp lệ thì đóng kết nối
        // $_SERVER['HTTP_ORIGIN'] biểu thị nguồn trang web nào đã khởi tạo kết nối websocket
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // trong onWebSocketConnect, $_GET $_SERVER có sẵn
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Khi workerman >= 5.0**
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

## Truyền dữ liệu nhị phân qua giao thức WebSocket

Giao thức websocket mặc định chỉ có thể truyền dữ liệu utf8, nếu muốn truyền dữ liệu nhị phân, vui lòng đọc phần sau.

Trong giao thức websocket, có một cờ hiệu trong tiêu đề giao thức để đánh dấu liệu truyền là dữ liệu nhị phân hay dữ liệu văn bản utf8, trình duyệt sẽ kiểm tra xem cờ hiệu và kiểu nội dung truyền có phù hợp không, nếu không phù hợp sẽ báo lỗi và ngắt kết nối.

Vì vậy, khi máy chủ gửi dữ liệu, cần thiết lập cờ hiệu này tùy theo loại dữ liệu truyền. Trong Workerman, nếu là văn bản utf8 thông thường, cần thiết lập (thường sẵn có giá trị này, thường không cần thiết lập lại)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Nếu là dữ liệu nhị phân, cần thiết lập
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Lưu ý**: Nếu không thiết lập $connection->websocketType, thì giá trị mặc định của $connection->websocketType là BINARY_TYPE_BLOB (nghĩa là loại văn bản utf8). Thông thường ứng dụng truyền là văn bản utf8, chẳng hạn dữ liệu json, nên không cần thiết lập $connection->websocketType. Chỉ khi truyền dữ liệu nhị phân (ví dụ dữ liệu hình ảnh, dữ liệu protobuffer), mới cần thiết lập thuộc tính này thành BINARY_TYPE_ARRAYBUFFER.

## Sử dụng workerman như một khách hàng WebSocket

Có thể sử dụng lớp [AsyncTcpConnection](../async-tcp-connection.md) cùng với [giao thức ws](about-ws.md) để làm cho workerman trở thành khách hàng websocket kết nối với máy chủ WebSocket từ xa, hoàn thành việc trò chuyện theo thời gian thực hai chiều.
