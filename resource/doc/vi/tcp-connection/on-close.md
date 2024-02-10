# onClose
## Giải thích:
```php
callback Connection::$onClose
```

Callback này có cùng tác dụng với callback [Worker::$onClose](../worker/on-close.md), khác biệt là chỉ áp dụng cho kết nối hiện tại, nghĩa là có thể đặt callback onClose cho một kết nối cụ thể.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Khi có sự kiện kết nối
$worker->onConnect = function(TcpConnection $connection)
{
    // Đặt callback onClose cho kết nối
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connection closed\n";
    };
};
// Chạy worker
Worker::runAll();
```

Mã trên có cùng tác dụng với đoạn mã dưới đây

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Đặt callback onClose cho tất cả kết nối
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// Chạy worker
Worker::runAll();
```
