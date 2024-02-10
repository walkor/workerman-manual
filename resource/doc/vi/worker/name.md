# Tên

## Mô tả:
```php
string Worker::$name
```

Đặt tên cho Worker hiện tại để dễ nhận biết tiến trình khi chạy lệnh status. Nếu không được đặt, mặc định là none.

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Đặt tên cho instance
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Chạy worker
Worker::runAll();
```
