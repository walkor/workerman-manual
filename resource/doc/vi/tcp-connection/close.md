# close
## Giải thích:
```php
void Connection::close(mixed $data = '')
```

Đóng kết nối một cách an toàn.
Gọi hàm close sẽ đợi cho đến khi dữ liệu trong bộ đệm gửi đi hoàn tất rồi mới đóng kết nối, và kích hoạt callback ```onClose``` của kết nối.

## Tham số

 ``` $data ```

Tham số tùy chọn, dữ liệu muốn gửi (nếu có chỉ định giao thức, thì sẽ tự động gọi phương thức mã hóa của giao thức để đóng gói dữ liệu ```$data```), sau khi gửi dữ liệu hoàn tất sẽ đóng kết nối và sau đó kích hoạt callback onClose.


## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Chạy worker
Worker::runAll();
```
