# maxPackageSize

## Giải thích:
```php
static int Connection::$defaultMaxPackageSize
```

Thuộc tính này là một thuộc tính tĩnh toàn cục, được sử dụng để đặt kích thước tối đa của gói dữ liệu mà mỗi kết nối có thể nhận. Nếu không được đặt, giá trị mặc định là 10MB. Nếu kích thước gói dữ liệu nhận được từ phía gửi (giá trị trả về từ phương thức input của lớp giao thức) lớn hơn ```Connection::$defaultMaxPackageSize```, thì dữ liệu sẽ bị coi là không hợp lệ và kết nối sẽ bị ngắt.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Đặt kích thước gói dữ liệu tối đa mà mỗi kết nối có thể nhận là 1024000 byte
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Chạy worker
Worker::runAll();
```
