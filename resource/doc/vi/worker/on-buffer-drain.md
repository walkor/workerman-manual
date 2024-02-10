# onBufferDrain
## Miêu tả:
```php
callback Worker::$onBufferDrain
```

Mỗi kết nối đều có một bộ đệm gửi ứng dụng riêng, kích thước bộ đệm được quyết định bởi ```TcpConnection::$maxSendBufferSize```, giá trị mặc định là 1MB và có thể thay đổi bằng cách thiết lập thủ công. Sau khi thay đổi, tất cả các kết nối sẽ áp dụng kích thước mới.

Callback này được kích hoạt sau khi tất cả dữ liệu trong bộ đệm gửi ứng dụng đã được gửi hoàn toàn. Thường được sử dụng cùng với `onBufferFull`, ví dụ như tạm dừng gửi dữ liệu đến đầu cuối khi `onBufferFull`, và tiếp tục gửi dữ liệu khi `onBufferDrain.



## Tham số của hàm callback

 ``` $connection ```

Đối tượng kết nối, tức là [thể hiện TcpConnection](../tcp-connection.md), được sử dụng để điều khiển kết nối của client, như [gửi dữ liệu](../tcp-connection/send.md), [đóng kết nối](../tcp-connection/close.md), v.v.


## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "Bộ đệm đầy và không gửi nữa\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "Bộ đệm đã trống và tiếp tục gửi\n";
};
// Chạy worker
Worker::runAll();
```

Lưu ý: Ngoài việc sử dụng hàm vô danh làm callback, bạn cũng có thể [tham khảo ở đây](../faq/callback_methods.md) để sử dụng cách viết callback khác.

## Xem thêm
onBufferFull: Khi bộ đệm gửi ứng dụng của kết nối đầy sẽ kích hoạt
