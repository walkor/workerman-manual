# onError
## Giải thích:
```php
callback Worker::$onError
```

Khi xảy ra lỗi trên kết nối của khách hàng, sự kiện này sẽ được kích hoạt.

Hiện tại có các loại lỗi sau:

1. Gọi Connection::send thất bại do kết nối của khách hàng bị ngắt kết nối (sau đó sự kiện onClose sẽ được kích hoạt) ``` (code:WORKERMAN_SEND_FAIL msg:client closed) ```

2. Sau khi kích hoạt onBufferFull (bộ đệm gửi đã đầy), vẫn gọi Connection::send và bộ đệm gửi vẫn đầy dẫn đến việc gửi thất bại (không kích hoạt sự kiện onClose)``` (code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package) ```

3. Khi kết nối không đồng bộ với AsyncTcpConnection thất bại (sau đó sự kiện onClose sẽ được kích hoạt) ``` (code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client trả về thông báo lỗi) ```


## Tham số của hàm gọi lại

 ``` $connection ```

Đối tượng kết nối, cụ thể [thực例 TcpConnection](../tcp-connection.md), được sử dụng để thao tác với kết nối khách hàng như [gửi dữ liệu](../tcp-connection/send.md), [đóng kết nối](../tcp-connection/close.md) và các thao tác khác.

 ``` $code ```

Mã lỗi

 ``` $msg ```

Thông báo lỗi


## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// Chạy worker
Worker::runAll();
```

Ghi chú: Ngoài việc sử dụng hàm vô danh làm hàm gọi lại, bạn cũng có thể [tham khảo ở đây](../faq/callback_methods.md) để sử dụng các cách viết hàm gọi lại khác.
