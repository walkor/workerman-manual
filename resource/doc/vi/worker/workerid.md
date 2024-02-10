# id
```（workerman >= 3.2.1）```

## Miêu tả:
```php
int Worker::$id
```

ID của quá trình worker hiện tại, có giá trị từ ```0``` đến ```$worker->count-1```.

Thuộc tính này rất hữu ích để phân biệt quá trình worker. Ví dụ, nếu một instance worker có nhiều quá trình và người phát triển chỉ muốn đặt bộ định thời trong một quá trình cụ thể, có thể làm điều này bằng cách nhận dạng ID của quá trình và đặt bộ định thời chỉ trong quá trình có số ID là 0 (xem ví dụ).

## Chú ý:

ID không thay đổi sau khi quá trình được khởi động lại.

Sự phân phối ID quá trình dựa trên từng instance worker. Mỗi instance worker bắt đầu phân phối ID quá trình của chính mình từ 0, vì vậy có thể có trùng lặp ID quá trình giữa các instance worker, nhưng một instance worker không bao giờ trùng lặp ID quá trình. Ví dụ như dưới đây:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// instance worker1 có 4 quá trình, ID quá trình sẽ lần lượt là 0, 1, 2, 3
$worker1 = new Worker('tcp://0.0.0.0:8585');
// Thiết lập khởi động 4 quá trình
$worker1->count = 4;
// Sau khi mỗi quá trình khởi động, in ID quá trình hiện tại là $worker1->id
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// instance worker2 có 2 quá trình, ID quá trình sẽ lần lượt là 0, 1
$worker2 = new Worker('tcp://0.0.0.0:8686');
// Thiết lập khởi động 2 quá trình
$worker2->count = 2;
// Sau khi mỗi quá trình khởi động, in ID quá trình hiện tại là $worker2->id
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Chạy worker
Worker::runAll();
```
Kết quả tương tự như sau:
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Chú ý: Trên hệ thống windows do không hỗ trợ việc thiết lập số lượng quá trình count, chỉ có một ID duy nhất là 0. Trên hệ thống windows không hỗ trợ khởi tạo hai Worker lắng nghe từ cùng một tệp, do đó, ví dụ này không thể chạy trên hệ thống windows.


## Ví dụ
Một instance worker có 4 quá trình, chỉ đặt bộ định thời trong quá trình có số ID là 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Chỉ đặt bộ định thời trong quá trình có số ID là 0, các quá trình khác 1, 2, 3 sẽ không đặt bộ định thời
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 quá trình worker, chỉ đặt bộ định thời trong quá trình số 0\n";
        });
    }
};
// Chạy worker
Worker::runAll();
```
