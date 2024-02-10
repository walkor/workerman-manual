# worker
## Miêu tả:
```php
Worker Connection::$worker
```

Thuộc tính này là thuộc tính chỉ đọc, nghĩa là một ví dụ về Worker Connection::$worker.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Khi một client gửi dữ liệu, chuyển tiếp đến tất cả các client khác đang được duy trì bởi quá trình hiện tại
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Chạy worker
Worker::runAll();
```
