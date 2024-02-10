# onBufferFull
## Giải thích:
```php
callback Worker::$onBufferFull
```

Mỗi kết nối đều có một bộ đệm gửi ở tầng ứng dụng riêng, nếu tốc độ nhận của client thấp hơn tốc độ gửi của server, dữ liệu sẽ được lưu trữ tạm thời trong bộ đệm ứng dụng và nếu bộ đệm đầy sẽ kích hoạt gọi lại `onBufferFull`.

Kích thước bộ đệm là [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md), giá trị mặc định là 1MB, có thể đặt kích thước bộ đệm động cho kết nối hiện tại ví dụ:
```php
// Đặt kích thước bộ đệm gửi cho kết nối hiện tại, tính bằng byte
$connection->maxSendBufferSize = 102400;
```
Cũng có thể sử dụng [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) để đặt kích thước mặc định cho tất cả kết nối, ví dụ:
```php
use Workerman\Connection\TcpConnection;
// Đặt kích thước mặc định cho bộ đệm gửi ở tầng ứng dụng cho tất cả kết nối, tính bằng byte
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```
Gọi lại này **có thể** được kích hoạt ngay sau khi gọi Connection::send, ví dụ như gửi dữ liệu lớn hoặc liên tục gửi dữ liệu nhanh tới đầu cuối do dữ liệu bị tích tụ trên bộ đệm gửi của kết nối tương ứng vì lý do mạng và khi vượt quá giới hạn ```TcpConnection::$maxSendBufferSize``` thì sẽ kích hoạt.

Khi có sự kiện onBufferFull xảy ra, người phát triển thường cần có biện pháp, ví dụ như tạm dừng gửi dữ liệu tới đầu cuối và chờ đợi dữ liệu trên bộ đệm gửi được gửi đi hết (sự kiện onBufferDrain).

Khi gọi Connection::send(`$A`) dẫn đến kích hoạt onBufferFull, bất kể dữ liệu `$A` gửi đi có lớn đến mức nào, ngay cả khi lớn hơn `TcpConnection::$maxSendBufferSize`, dữ liệu cần gửi lần này vẫn sẽ được đặt vào bộ đệm gửi. Điều đó có nghĩa là bộ đệm gửi thực tế có thể lớn hơn rất nhiều so với `TcpConnection::$maxSendBufferSize`. Khi dữ liệu trên bộ đệm gửi đã lớn hơn giới hạn `TcpConnection::$maxSendBufferSize`, nếu tiếp tục gửi dữ liệu Connection::send(`$B`), dữ liệu `$B` này sẽ không được đặt vào bộ đệm gửi mà sẽ bị loại bỏ và kích hoạt `onError`.

Tóm lại, chỉ cần bộ đệm gửi chưa đầy, ngay cả chỉ còn một byte trống, gọi Connection::send(```$A```) chắc chắn sẽ đặt `A` vào bộ đệm gửi, nếu sau khi đặt vào bộ đệm gửi và kích thước bộ đệm gửi vượt quá giới hạn `TcpConnection::$maxSendBufferSize`, sẽ kích hoạt gọi lại onBufferFull.

## Tham số của hàm gọi lại

 ``` $connection ```

Đối tượng kết nối, cụ thể là [TcpConnection instance](../tcp-connection.md), được sử dụng để thao tác với kết nối client, như [gửi dữ liệu](../tcp-connection/send.md), [đóng kết nối](../tcp-connection/close.md) và các thao tác khác.


## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
// Khởi chạy worker
Worker::runAll();
```

Ghi chú: Ngoài việc sử dụng hàm vô danh làm lời gọi lại, có thể [tham khảo ở đây](../faq/callback_methods.md) để sử dụng cách viết lời gọi lại khác.

## Xem thêm
onBufferDrain khi dữ liệu trên bộ đệm gửi ở tầng ứng dụng của kết nối được gửi hết sẽ kích hoạt
