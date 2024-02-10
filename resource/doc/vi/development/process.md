# Quy trình cơ bản
(Trong trường hợp của một máy chủ phòng chat Websocket đơn giản)

#### 1. Tạo thư mục dự án ở bất kỳ vị trí nào
Ví dụ: SimpleChat/
Chạy lệnh `composer require workerman/workerman` trong thư mục đó.

#### 2. Nhập `vendor/autoload.php` (sau khi cài đặt bằng composer)
Tạo `start.php`, nhập `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. Chọn giao thức
Ở đây chúng ta chọn giao thức văn bản (là một giao thức do WorkerMan tự định nghĩa, có định dạng dữ liệu là văn bản + dòng xuống hàng)

(Hiện tại WorkerMan hỗ trợ giao thức HTTP, Websocket, giao thức văn bản. Nếu cần sử dụng giao thức khác, vui lòng tham khảo chương về giao thức để phát triển giao thức riêng của mình)

#### 4. Viết tệp khởi đầu theo nhu cầu
Ví dụ dưới đây là một tệp khởi đầu đơn giản của một phòng chat.

SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// Khi người dùng kết nối, gắn uid cho kết nối và thông báo cho tất cả người dùng khác
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // Gắn một uid cho kết nối này
    $connection->uid = ++$global_uid;
}

// Khi người dùng gửi tin nhắn, chuyển tiếp tin nhắn cho tất cả mọi người
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] said: $data");
    }
}

// Khi người dùng ngắt kết nối, phát sóng cho tất cả người dùng khác
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] logout");
    }
}

// Tạo một Worker với giao thức văn bản lắng nghe cổng 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// Chỉ kích hoạt 1 tiến trình, điều này làm cho việc truyền dữ liệu giữa các đối tượng kết nối dễ dàng hơn
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();
```

#### 5. Kiểm thử
Giao thức văn bản có thể được kiểm thử bằng lệnh telnet
```shell
telnet 127.0.0.1 2347
```
