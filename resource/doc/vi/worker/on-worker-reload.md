# onWorkerReload

Yêu cầu ```（workerman >= 3.2.5）```
## Mô tả:

```php
callback Worker::$onWorkerReload
```

Tính năng này ít được sử dụng.

Thiết lập callback mà Worker sẽ thực hiện sau khi nhận được tín hiệu reload.

Có thể sử dụng callback onWorkerReload để thực hiện nhiều công việc, ví dụ như để tải lại tệp cấu hình kinh doanh mà không cần khởi động lại quá trình.

**Chú ý**:

Hành động mặc định của tiểu trình sau khi nhận tín hiệu reload là thoát và khởi động lại để có thể tiến hành tải lại mã kinh doanh mới. Do đó, là hiện tượng bình thường nếu tiểu trình sau khi thực hiện callback onWorkerReload sẽ thoát ngay lập tức.

Nếu sau khi nhận tín hiệu reload bạn chỉ muốn để tiểu trình thực hiện onWorkerReload mà không muốn thoát, bạn có thể thiết lập thuộc tính reloadable của Worker instance tương ứng thành false khi khởi tạo Worker instance.


## Tham số của hàm callback

 ``` $worker ```

Được hiểu là đối tượng Worker



## Ví dụ


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Đặt reloadable thành false, nghĩa là tiểu trình sau khi nhận tín hiệu reload sẽ không khởi động lại
$worker->reloadable = false;
// Sau khi reload, thông báo với tất cả các kết nối rằng máy chủ đã thực hiện reload
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Chạy worker
Worker::runAll();
```

Ghi chú: Ngoài việc sử dụng hàm ẩn danh làm callback, bạn cũng có thể [tham khảo ở đây](../faq/callback_methods.md) để sử dụng cách viết callback khác.
