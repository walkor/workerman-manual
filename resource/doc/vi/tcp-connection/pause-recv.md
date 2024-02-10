# pauseRecv
## Giải thích:
```php
void Connection::pauseRecv(void)
```

Dừng việc nhận dữ liệu của kết nối hiện tại. Callback onMessage của kết nối sẽ không được kích hoạt. Phương pháp này rất hữu ích cho việc kiểm soát lưu lượng tải lên.

## Tham số

Không có tham số

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Động viên cho đối tượng connection thêm một thuộc tính động, để lưu trữ số lượng yêu cầu hiện tại đến từ kết nối này
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Sau mỗi kết nối nhận được 100 yêu cầu thì không nhận thêm dữ liệu nữa
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Chạy worker
Worker::runAll();
```

## Xem thêm
void Connection::resumeRecv(void) - Làm cho đối tượng kết nối tương ứng tiếp tục nhận dữ liệu
