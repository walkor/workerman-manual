# on
**``` (Yêu cầu phiên bản Workerman >= 3.3.0) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
Đăng ký sự kiện ```$event_name``` và đăng ký hàm gọi lại ```$callback_function``` khi sự kiện xảy ra.

## Tham số của hàm gọi lại

``` $event_name ```

Tên của sự kiện đã đăng ký, có thể là chuỗi bất kỳ.

``` $callback_function ```

Hàm gọi lại khi sự kiện xảy ra. Nguyên mẫu của hàm là ```callback_function(mixed $event_data)```. ```$event_data``` là dữ liệu sự kiện được truyền khi phát sự kiện.

Chú ý:

Nếu cùng một sự kiện được đăng ký với hai hàm gọi lại, hàm gọi lại sau sẽ ghi đè lên hàm gọi lại trước đó.

## Ví dụ
Worker đa tiến trình (nhiều máy chủ), một khách hàng gửi tin nhắn và phát sóng đến tất cả các khách hàng

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Khởi tạo một máy chủ Channel
$channel_server = new Channel\Server('0.0.0.0', 2206);

// Máy chủ websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// Mỗi khi một quá trình worker khởi động
$worker->onWorkerStart = function($worker)
{
    // Kết nối từ Client của Channel đến máy chủ Channel
    Channel\Client::connect('127.0.0.1', 2206);
    // Đăng ký sự kiện broadcast và đăng ký hàm gọi lại sự kiện
    Channel\Client::on('broadcast', function($event_data) use ($worker){
        // Phát tin nhắn broadcast đến tất cả các kết nối của quá trình worker hiện tại
        foreach ($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // Sử dụng dữ liệu từ client như dữ liệu sự kiện
   $event_data = $data;
   // Phát sự kiện broadcast đến tất cả các quá trình worker
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Kiểm tra**

Mở trình duyệt chrome, nhấn F12 để mở bảng điều khiển gỡ lỗi, trong mục Console, nhập (hoặc đặt đoạn mã sau vào trang HTML và chạy bằng JavaScript)

Kết nối nhận tin nhắn
```javascript
// Thay đổi 127.0.0.1 thành địa chỉ IP thực của Workerman
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e){
    alert("Nhận được tin nhắn từ máy chủ: " + e.data);
};
```

Phát tin nhắn broadcast
```shell
ws.send('Xin chào thế giới');
```
