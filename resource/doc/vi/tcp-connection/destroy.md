# destroy
## Giải thích:
```php
void Connection::destroy()
```

Đóng kết nối ngay lập tức.

Khác biệt so với close ở chỗ là sau khi gọi destroy, ngay cả khi bộ đệm gửi của kết nối này còn dữ liệu chưa gửi đi cho bên kia, kết nối cũng sẽ bị đóng ngay lập tức và kích hoạt callback ```onClose``` của kết nối ngay lập tức.

## Tham số

Không có tham số


## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // nếu có lỗi
    $connection->destroy();
};
// chạy worker
Worker::runAll();
```
