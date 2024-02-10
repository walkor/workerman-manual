# Đếm

## Miêu tả:
```php
int Worker::$count
```

Thiết lập số lượng tiến trình mà Worker hiện tại sẽ khởi động, mặc định là 1 nếu không được thiết lập. 

Vui lòng xem chi tiết để thiết lập số lượng tiến trình tại [**đây**](../faq/processes-count.md).

Lưu ý: Thuộc tính này phải được thiết lập trước khi chạy ```Worker::runAll();``` để có hiệu lực. Tính năng này không hỗ trợ trên hệ điều hành Windows.

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Khởi động 8 tiến trình, cùng lúc lắng nghe cổng 8484, cung cấp dịch vụ theo giao thức websocket
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Chạy worker
Worker::runAll();
```
