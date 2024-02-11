# pidFile
## Giải thích:
```php
static string Worker::$pidFile
```

Nếu không có nhu cầu đặc biệt, đề xuất không nên thiết lập thuộc tính này.

Thuộc tính này là một thuộc tính tĩnh toàn cục, được sử dụng để thiết lập đường dẫn tệp pid của Workerman.

Thiết lập này rất hữu ích trong việc theo dõi, ví dụ như đặt tệp pid của Workerman vào một thư mục cố định để giúp các phần mềm theo dõi đọc tệp pid và theo dõi trạng thái của tiến trình Workerman.

Nếu không thiết lập, Workerman mặc định sẽ tự động tạo một tệp pid ở cùng cấp độ thư mục với thư mục Workerman (lưu ý rằng trước phiên bản 3.2.3, mặc định là ```sys_get_temp_dir()```), và để tránh việc khởi động nhiều thể hiện Workerman dẫn đến xung đột pid, Workerman sẽ bao gồm đường dẫn hiện tại của Workerman trong tệp pid được tạo.

Lưu ý: Thuộc tính này chỉ có hiệu lực khi được thiết lập trước khi chạy ```Worker::runAll();```. Hệ điều hành windows không hỗ trợ tính năng này.

## Ví dụ

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Chạy worker
Worker::runAll();
```
