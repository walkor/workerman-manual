# getRemoteIp
## Giải thích:
```php
string Connection::getRemoteIp()
```

Lấy địa chỉ IP của khách hàng kết nối

## Tham số
Không có tham số

## Ví dụ
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "kết nối mới từ địa chỉ IP " . $connection->getRemoteIp() . "\n";
};
// Chạy worker
Worker::runAll();
```
