# maxSendBufferSize
## Giải thích:
```php
int Connection::$maxSendBufferSize
```

Mỗi kết nối đều có một bộ đệm gửi ở tầng ứng dụng riêng biệt, nếu tốc độ nhận của client chậm hơn tốc độ gửi của server, dữ liệu sẽ được lưu trữ tạm thời trong bộ đệm ứng dụng đợi để gửi đi.

Thuộc tính này được sử dụng để thiết lập kích thước của bộ đệm gửi ở tầng ứng dụng của kết nối hiện tại. Nếu không thiết lập, mặc định là [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1MB).

Thuộc tính này ảnh hưởng đến callback [onBufferFull](../worker/on-buffer-full.md).



## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Thiết lập kích thước bộ đệm gửi ở tầng ứng dụng của kết nối hiện tại là 102400 byte
    $connection->maxSendBufferSize = 102400;
};
// Chạy worker
Worker::runAll();
```
