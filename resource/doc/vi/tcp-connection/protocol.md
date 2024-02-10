# giao thức

## Miêu tả:
```php
string Connection::$protocol
```

Thiết lập lớp giao thức cho kết nối hiện tại.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // Khi gửi, tự động gọi $connection->protocol::encode() để đóng gói dữ liệu trước khi gửi
    $connection->send("hello");
};
// Chạy worker
Worker::runAll();
```
