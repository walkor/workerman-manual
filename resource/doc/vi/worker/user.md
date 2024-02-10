# Người dùng

## Giải thích:
```php
string Worker::$user
```

Đặt Worker hiện tại chạy dưới quyền của người dùng nào. Thuộc tính này chỉ có thể có hiệu lực khi người dùng hiện tại là root. Nếu không được thiết lập, mặc định sẽ chạy dưới quyền của người dùng hiện tại.

Đề xuất thiết lập ```$user``` là một người dùng có quyền hạn thấp, ví dụ như www-data, apache, nobody, vv.

Lưu ý: Thuộc tính này phải được thiết lập trước khi chạy ```Worker::runAll();```. Hệ thống Windows không hỗ trợ tính năng này.

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Thiết lập người dùng chạy worker
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Đang khởi động Worker...\n";
};
// Chạy worker
Worker::runAll();
```
