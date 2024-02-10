# reloadable
## Giải thích:
```php
bool Worker::$reloadable
```

Khi thực hiện `php start.php reload`, một tín hiệu reload (SIGUSR1) sẽ được gửi đến tất cả các tiến trình con.

Khi tiến trình con nhận được tín hiệu reload, nó sẽ tự động thoát và sau đó tiến trình chính sẽ tự động khởi động một tiến trình mới, thường được sử dụng để cập nhật mã kinh doanh.

Khi thuộc tính $reloadable của tiến trình là false, sau khi nhận tín hiệu reload chỉ kích hoạt [onWorkerReload](on-worker-reload.md) và không khởi động lại tiến trình hiện tại.

Ví dụ, trong mô hình Gateway/Worker, tiến trình gateway chịu trách nhiệm duy trì kết nối khách hàng và tiến trình worker chịu trách nhiệm xử lý yêu cầu. Thiết lập thuộc tính reloadable của tiến trình gateway là false sẽ cho phép cập nhật mã kinh doanh mà không cần ngắt kết nối của khách hàng.

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Thiết lập có khởi động lại sau khi nhận tín hiệu reload hay không
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Chạy worker
Worker::runAll();
```
