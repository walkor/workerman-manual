# resumeRecv
## Giải thích:
```php
void Connection::resumeRecv(void)
```

Cho phép kết nối hiện tại tiếp tục nhận dữ liệu. Phương thức này được sử dụng cùng với Connection::pauseRecv, rất hữu ích cho việc kiểm soát lưu lượng tải lên.

## Tham số

Không có tham số

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Thêm một thuộc tính động vào đối tượng connection để lưu trữ số lượng yêu cầu được gửi từ kết nối hiện tại
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Sau mỗi kết nối tiếp nhận 100 yêu cầu thì không tiếp nhận dữ liệu nữa
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // Sau 30 giây, tiếp tục tiếp nhận dữ liệu
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Chạy worker
Worker::runAll();
```

## Xem thêm
void Connection::pauseRecv(void) Dừng việc nhận dữ liệu từ đối tượng kết nối tương ứng
