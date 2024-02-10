# stdoutFile
## Giải thích:
```php
static string Worker::$stdoutFile
```

Thuộc tính này là một thuộc tính tĩnh toàn cục. Nếu chạy ở chế độ daemon (bằng cách sử dụng ```-d```), tất cả đầu ra đến terminal (echo var_dump vv) sẽ được chuyển hướng vào tập tin được chỉ định trong stdoutFile.

Nếu không được thiết lập và chạy ở chế độ daemon, tất cả đầu ra terminal sẽ được chuyển hướng đến `/dev/null` (nghĩa là mặc định hủy bỏ tất cả đầu ra)

> Lưu ý: `/dev/null` là một tập tin đặc biệt trên hệ điều hành Linux, thực tế là một hố đen, mọi dữ liệu được ghi vào tập tin này sẽ bị hủy bỏ. Nếu không muốn hủy bỏ đầu ra, có thể thiết lập `Worker::$stdoutFile` thành một đường dẫn tập tin bình thường.

> Lưu ý: Thuộc tính này phải được thiết lập trước khi chạy ```Worker::runAll();``` mới có hiệu lực. Hệ điều hành Windows không hỗ trợ tính năng này.

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Tất cả đầu ra in ra sẽ được lưu vào tệp /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Chạy worker
Worker::runAll();
```
