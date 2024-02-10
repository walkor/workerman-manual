# ống nước
## Mô tả:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## Tham số
Chuyển luồng dữ liệu của kết nối hiện tại vào kết nối mục tiêu. Có sẵn kiểm soát lưu lượng. Phương pháp này rất hữu ích cho việc cung cấp dịch vụ proxy TCP

## Ví dụ về dịch vụ proxy TCP

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// Sau khi kết nối TCP được thiết lập
$worker->onConnect = function(TcpConnection $connection)
{
    // Thiết lập kết nối không đồng bộ tới cổng 80 cục bộ
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Thiết lập việc chuyển dữ liệu của kết nối khách hàng hiện tại đến kết nối cổng 80
    $connection->pipe($connection_to_80);
    // Thiết lập việc chuyển dữ liệu trả về từ kết nối cổng 80 đến kết nối khách hàng
    $connection_to_80->pipe($connection);
    // Kết nối không đồng bộ được thực hiện
    $connection_to_80->connect();
};

// Chạy worker
Worker::runAll();
```
