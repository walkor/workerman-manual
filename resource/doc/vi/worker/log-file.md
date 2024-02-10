# Tập tin nhật ký
## Mô tả:
```php
static string Worker::$logFile
```

Dùng để chỉ định vị trí tập tin nhật ký cho Workerman.

Tập tin này ghi lại các sự kiện liên quan đến Workerman, bao gồm việc khởi động, dừng lại, v.v.

Nếu không thiết lập, tên tập tin mặc định là workerman.log, vị trí tập tin là ở cấp độ trên của thư mục Workerman.

**Lưu ý:**

Tập tin nhật ký này chỉ ghi lại các sự kiện liên quan đến việc khởi động và dừng của Workerman, không bao gồm bất kỳ nhật ký kinh doanh nào.

Các nhật ký kinh doanh có thể được thực hiện bằng cách sử dụng hàm [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) hoặc [error_log](https://php.net/manual/zh/function.error-log.php) hoặc các phương pháp tự thiết lập.

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Chạy worker
Worker::runAll();
```
