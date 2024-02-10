# onConnect
## Mô tả:
```php
callback Worker::$onConnect
```

Khi kết nối giữa máy khách và Workerman được thiết lập (sau khi hoàn tất ba bước TCP) sẽ kích hoạt hàm gọi lại. Mỗi kết nối chỉ kích hoạt một lần hàm gọi lại ```onConnect```.

Lưu ý: Sự kiện onConnect chỉ đại diện cho việc máy khách và Workerman đã hoàn tất ba bước TCP, lúc này máy khách chưa gửi bất kỳ dữ liệu nào, vậy nên ngoài việc sử dụng ```$connection->getRemoteIp()``` để lấy địa chỉ IP của đối tác, không có thông tin nào khác để xác định máy khách, do đó trong sự kiện onConnect không thể xác định ai là đối tác. Để biết đối tác là ai, cần máy khách gửi dữ liệu xác thực, chẳng hạn như một token hoặc tên người dùng và mật khẩu, và xác thực trong [hàm gọi lại onMessage](on-message.md).

Do udp không có kết nối, vì vậy khi sử dụng udp, sẽ không kích hoạt hàm gọi lại onConnect, cũng không kích hoạt hàm gọi lại onClose.

## Tham số của hàm gọi lại

 ``` $connection ```

Đối tượng kết nối, cụ thể là [Thực例 Kết nối TCP](../tcp-connection.md), được sử dụng để thao tác với kết nối máy khách, như [gửi dữ liệu](../tcp-connection/send.md), [đóng kết nối](../tcp-connection/close.md), v.v.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "Kết nối mới từ địa chỉ IP " . $connection->getRemoteIp() . "\n";
};
// Khởi chạy worker
Worker::runAll();
```

Ghi chú: Ngoài việc sử dụng hàm không tên làm hàm gọi lại, còn có thể [tham khảo ở đây](../faq/callback_methods.md) cách viết hàm gọi lại khác.
