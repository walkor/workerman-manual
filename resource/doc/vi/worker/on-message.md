# onMessage
## Giải thích:
```php
callback Worker::$onMessage
```

Khi dữ liệu được gửi từ client thông qua kết nối (Workerman nhận dữ liệu) thì sẽ kích hoạt hàm gọi lại này.

## Tham số của hàm gọi lại

``` $connection ```

Đối tượng kết nối, tức là [thực例 TcpConnection](../tcp-connection.md), được sử dụng để thao tác với kết nối của client, như [gửi dữ liệu](../tcp-connection/send.md), [đóng kết nối](../tcp-connection/close.md) và những thao tác khác.

``` $data ```

Dữ liệu được gửi từ kết nối của client, nếu Worker đã chỉ định một giao thức, thì $data sẽ là dữ liệu đã được decode (giải mã) tương ứng với giao thức đó. Loại dữ liệu phụ thuộc vào cách thức implement của hàm `decode()` của giao thức, `websocket`, `text`, `frame` sẽ là chuỗi ký tự, giao thức HTTP sẽ là đối tượng [`Workerman\Protocols\Http\Request`](../http/request.md).

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// Chạy worker
Worker::runAll();
```

Gợi ý: Ngoài việc sử dụng hàm vô danh làm hàm gọi lại, còn có thể tham khảo [ở đây](../faq/callback_methods.md) để sử dụng cách viết hàm gọi lại khác.
