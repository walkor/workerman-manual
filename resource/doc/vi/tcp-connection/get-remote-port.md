# Phương thức getRemotePort
## Mô tả:
```php
int Connection::getRemotePort()
```

Trả về cổng của máy khách kết nối này.

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
    echo "Kết nối mới từ địa chỉ ".
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Chạy worker
Worker::runAll();
```
