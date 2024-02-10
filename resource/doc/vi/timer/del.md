# del
```php
boolean \Workerman\Timer::del(int $timer_id)
```
Xóa một bộ định thời cụ thể

### Tham số
``` timer_id ```

Id của định thời, tức là số nguyên trả về từ giao diện add

### Giá trị trả về
boolean


### Ví dụ
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Mở nhiều quy trình để chạy công việc định kỳ, lưu ý vấn đề đồng thời đa quy trình
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Chạy mỗi 2 giây một lần
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // Chạy một công việc một lần sau 20 giây, xóa việc định kỳ chạy mỗi 2 giây
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Chạy worker
Worker::runAll();
```

### Ví dụ (Xóa bộ định thời trong callback của bộ định thời)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Chú ý rằng, trong callback sử dụng id bộ định thời hiện tại phải sử dụng tham chiếu (&)
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // Chạy 10 lần rồi xóa bộ định thời
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Chạy worker
Worker::runAll();
```
