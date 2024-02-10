# onWorkerStart
## Giải thích:
```php
callback Worker::$onWorkerStart
```

Thiết lập hàm gọi lại cho quá trình khởi động con Worker, mỗi lần một con Worker khởi động thì hàm sẽ được thực thi.

Chú ý: onWorkerStart được thực thi khi một con Worker khởi động, nếu có nhiều con Worker (`$worker->count > 1`), mỗi con Worker thực thi một lần, tổng cộng sẽ thực thi `$worker->count` lần.


## Tham số của hàm gọi lại

 ``` $worker ```

Đó là đối tượng Worker



## Ví dụ


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Con Worker đang khởi động...\n";
};
// Chạy worker
Worker::runAll();
```

Lưu ý: Ngoài việc sử dụng hàm vô danh như là hàm gọi lại, bạn cũng có thể [tham khảo ở đây](../faq/callback_methods.md) để sử dụng cách viết mã hàm gọi lại khác.
