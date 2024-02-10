# Cách thông báo tin nhắn tự động

1. Có thể sử dụng bộ hẹn giờ để tự động gửi dữ liệu
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// Sau khi quá trình khởi động, gửi dữ liệu định kỳ cho máy khách
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. Thông báo workerman để gửi dữ liệu khi có sự kiện xảy ra trong dự án khác
Xem thêm tại [Câu hỏi phổ biến - Gửi dữ liệu trong dự án khác](push-in-other-project.md)
