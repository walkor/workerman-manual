# kết nối
## Miêu tả:
```php
mảng Worker::$connections
```

Định dạng
```php
mảng(id=>kết nối, id=>kết nối, ...)
```

Thuộc tính này lưu trữ tất cả các đối tượng kết nối của **quá trình hiện tại**, trong đó id là số thứ tự của kết nối, chi tiết xem trong tài liệu [Thuộc tính id của TcpConnection](../tcp-connection/id.md).

Biến ```$connections``` rất hữu ích trong việc phát sóng hoặc lấy đối tượng kết nối dựa trên id.

Nếu biết số thứ tự của kết nối là ```$id```, có thể dễ dàng lấy đối tượng kết nối tương ứng thông qua ```$worker->connections[$id]```, từ đó điều khiển kết nối socket tương ứng, ví dụ như gửi dữ liệu bằng ```$worker->connections[$id]->send('...')```.

Lưu ý: Nếu kết nối đóng (kích hoạt onClose), thì ```connection``` tương ứng sẽ bị xóa khỏi mảng ```$connections```.

Lưu ý: Không nên thay đổi thuộc tính này, hoặc có thể gây ra tình huống không thể dự đoán.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// Thiết lập một bộ đếm thời gian khi quá trình bắt đầu, gửi dữ liệu đến tất cả kết nối khách hàng theo chu kỳ
$worker->onWorkerStart = function($worker)
{
    // Theo chu kỳ, mỗi 10 giây
    Timer::add(10, function()use($worker)
    {
        // Duyệt qua tất cả kết nối khách hàng hiện tại của quá trình, gửi thời gian hiện tại của máy chủ
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Chạy worker
Worker::runAll();
```
