# Ví dụ phát triển đơn giản

## Cài đặt

**Cài đặt workerman**
Chạy lệnh sau trong thư mục trống
`composer require workerman/workerman`

## Ví dụ 1: Sử dụng giao thức HTTP để cung cấp dịch vụ Web

**Tạo tệp start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Tạo một Worker lắng nghe cổng 2345, sử dụng giao thức http
$http_worker = new Worker("http://0.0.0.0:2345");

// Khởi động 4 tiến trình cung cấp dịch vụ
$http_worker->count = 4;

// Khi nhận data từ trình duyệt, trả về "hello world" cho trình duyệt
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Gửi "hello world" đến trình duyệt
    $connection->send('hello world');
};

// Chạy worker
Worker::runAll();
```

**Chạy từ dòng lệnh (người dùng Windows sử dụng [dòng lệnh cmd](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn) tương tự)**
```shell
php start.php start
```

**Kiểm tra**
Giả sử IP máy chủ là 127.0.0.1

Truy cập url http://127.0.0.1:2345 trên trình duyệt

 **Chú ý:**

1. Nếu gặp sự cố không thể truy cập, vui lòng xem [Lý do thất bại kết nối từ máy khách](../faq/client-connect-fail.md) để xác định nguyên nhân.
2. Máy chủ sử dụng giao thức http, chỉ có thể truyền thông qua giao thức http, không thể truyền thông trực tiếp qua giao thức websocket hoặc giao thức khác.

## Ví dụ 2: Sử dụng giao thức WebSocket để cung cấp dịch vụ

**Tạo tệp ws_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Lưu ý: Ở đây khác với ví dụ trước, sử dụng giao thức websocket
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Khởi động 4 tiến trình cung cấp dịch vụ
$ws_worker->count = 4;

// Khi nhận data từ client, trả về "hello $data" cho client
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Gửi "hello $data" đến client
    $connection->send('hello ' . $data);
};

// Chạy worker
Worker::runAll();
```

**Chạy từ dòng lệnh**
```shell
php ws_test.php start
```

**Kiểm tra**

Mở trình duyệt Chrome, nhấn F12 để mở bảng điều khiển gỡ lỗi, nhập (hoặc chèn mã dưới đây vào trang html và chạy bằng javascript)

```javascript
// Giả sử IP máy chủ là 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Kết nối thành công");
    ws.send('tom');
    alert("Gửi một chuỗi 'tom' đến máy chủ");
};
ws.onmessage = function(e) {
    alert("Nhận thông điệp từ máy chủ: " + e.data);
};
```

  **Chú ý:**

1. Nếu gặp sự cố không thể truy cập, vui lòng xem [Mục phổ biến về Lỗi kết nối](../faq/client-connect-fail.md) để xác định nguyên nhân.
2. Máy chủ sử dụng giao thức websocket, chỉ có thể truyền thông qua giao thức websocket, không thể truyền thông trực tiếp qua giao thức http hoặc giao thức khác.

## Ví dụ 3: Truyền dữ liệu trực tiếp bằng giao thức TCP

**Tạo tệp tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Tạo một Worker lắng nghe cổng 2347, không sử dụng bất kỳ giao thức ứng dụng nào
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Khởi động 4 tiến trình cung cấp dịch vụ
$tcp_worker->count = 4;

// Khi client gửi dữ liệu đến
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Gửi "hello $data" đến client
    $connection->send('hello ' . $data);
};

// Chạy worker
Worker::runAll();
```

**Chạy từ dòng lệnh**

```shell
php tcp_test.php start
```

**Kiểm tra: Chạy từ dòng lệnh**
(Dưới đây là hiệu ứng dòng lệnh Linux, khác với hiệu ứng dòng lệnh trên Windows)
```shell
telnet 127.0.0.1 2347
Đang thử kết nối tới 127.0.0.1...
Đã kết nối tới 127.0.0.1.
Ký tự thoát là '^]'.
tom
hello tom
```

**Chú ý:**

1. Nếu gặp sự cố không thể truy cập, vui lòng xem [Mục phổ biến về Lỗi kết nối](../faq/client-connect-fail.md) để xác định nguyên nhân.
2. Máy chủ sử dụng giao thức TCP thuần túy, không thể truyền thông trực tiếp qua giao thức websocket, http hoặc giao thức khác.
