# Daemonize
## Giải thích:
```php
static bool Worker::$daemonize
```

Thuộc tính này là một thuộc tính tĩnh toàn cục, đại diện cho việc chạy ở chế độ daemon (tiến trình bảo vệ). Nếu lệnh khởi động sử dụng tham số ```-d```, thì thuộc tính này sẽ tự động được đặt thành true. Bạn cũng có thể thiết lập bằng mã nguồn.

Chú ý: Thuộc tính này phải được thiết lập trước khi chạy ```Worker::runAll();``` để có hiệu lực. Hệ điều hành Windows không hỗ trợ tính năng này.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Chạy worker
Worker::runAll();
```
