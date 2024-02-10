# onClose
## Giải thích:
```php
callback Worker::$onClose
```

Khi kết nối của client với Workerman bị ngắt, hàm callback này sẽ được kích hoạt. Không quan trọng là kết nối bị ngắt bằng cách nào, khi kết nối bị ngắt, ```onClose``` sẽ được kích hoạt. Mỗi kết nối chỉ được kích hoạt ```onClose``` một lần.

Lưu ý: Nếu kết nối bị ngắt do mất kết nối mạng hoặc mất điện và không thể gửi gói tin TCP fin cho Workerman kịp thời, Workerman sẽ không biết được kết nối đã bị ngắt và cũng không thể kích hoạt ```onClose``` kịp thời. Trường hợp này cần giải quyết bằng cách sử dụng cơ chế heartbeat ở tầng ứng dụng. Hiện thực của heartbeat kết nối trong Workerman được mô tả tại [đây](../faq/heartbeat.md). Nếu sử dụng framework GatewayWorker, bạn có thể sử dụng trực tiếp cơ chế heartbeat của framework này, xem thêm tại [đây](https://doc2.workerman.net/heartbeat.html).

Do UDP không phải là kết nối, nên khi sử dụng UDP sẽ không kích hoạt hàm callback onClose và onConnect.

## Tham số của hàm callback

 ``` $connection ```

Đối tượng kết nối, cụ thể là [TcpConnection instance](../tcp-connection.md), dùng để thao tác với kết nối của client như [gửi dữ liệu](../tcp-connection/send.md), [đóng kết nối](../tcp-connection/close.md), v.v.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "Kết nối đã đóng\n";
};
// Chạy worker
Worker::runAll();
```

Lưu ý: Ngoài việc sử dụng hàm vô danh làm callback, bạn cũng có thể [tham khảo tại đây](../faq/callback_methods.md) để sử dụng cách viết callback khác.
