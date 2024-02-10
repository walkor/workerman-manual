# Gửi
## Mô tả:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Gửi dữ liệu đến máy khách

## Tham số

 ``` $data ```

Dữ liệu cần gửi, nếu giao thức được chỉ định khi khởi tạo lớp Worker thì phương thức encode của giao thức sẽ tự động được gọi, hoàn thành công việc đóng gói theo giao thức trước khi gửi đến máy khách

 ``` $raw ```
 
Có gửi dữ liệu gốc, tức là không gọi phương thức encode của giao thức, mặc định là false, tức là tự động gọi phương thức encode của giao thức

## Giá trị trả về

true: Dữ liệu đã được ghi thành công vào bộ đệm gửi của socket tầng hệ điều hành của kết nối

null: Dữ liệu đã được ghi vào bộ đệm gửi tầng ứng dụng của kết nối, đợi để ghi vào bộ đệm gửi tầng socket của hệ điều hành

false: Gửi thất bại, lý do có thể là kết nối của máy khách đã đóng, hoặc bộ đệm gửi tầng ứng dụng của kết nối đã đầy

## Lưu ý
Khi gửi trả về ```true```, chỉ có nghĩa là dữ liệu đã được ghi thành công vào bộ đệm gửi của socket kết nối, và không có nghĩa là dữ liệu đã được gửi thành công đến bộ đệm nhận của socket bên đối diện, cũng không có nghĩa là ứng dụng bên đối diện đã đọc dữ liệu từ bộ đệm nhận của socket cục bộ. **Tuy nhiên, ngay cả khi gửi không trả về false và mạng vẫn chưa bị ngắt kết nối, và máy khách nhận đúng, dữ liệu về cơ bản có thể xem là đã gửi thành công đến bên kia.**

Do dữ liệu bộ đệm gửi của socket được gửi bất đồng bộ đến bên đối diện bởi hệ điều hành, hệ điều hành không cung cấp cơ chế xác nhận tương ứng cho tầng ứng dụng, vì vậy **tầng ứng dụng** không thể biết được khi nào dữ liệu trong bộ đệm gửi của socket bắt đầu gửi đi, **tầng ứng dụng** cũng không biết được liệu dữ liệu trong bộ đệm gửi của socket đã gửi thành công hay không. Do các lý do trên, workerman không thể cung cấp trực tiếp giao diện xác nhận tin nhắn.

Nếu kinh doanh cần đảm bảo rằng mỗi tin nhắn đều được máy khách nhận được, có thể thêm một cơ chế xác nhận trong kinh doanh. Cơ chế xác nhận có thể khác nhau tùy thuộc vào mô hình kinh doanh cụ thể, ngay cả khi cùng là mô hình kinh doanh, cơ chế xác nhận cũng có thể có nhiều phương pháp.

Ví dụ hệ thống trò chuyện có thể sử dụng cơ chế xác nhận như sau. Lưu mỗi tin nhắn vào cơ sở dữ liệu, mỗi tin nhắn có một trường xác định là đã đọc hay chưa. Khi máy khách nhận được một tin nhắn, máy khách gửi một gói xác nhận cho máy chủ và máy chủ đánh dấu tin nhắn tương ứng là đã đọc. Khi máy khách kết nối đến máy chủ (thường là đăng nhập người dùng hoặc kết nối lại sau khi mất kết nối), máy chủ tra cứu cơ sở dữ liệu xem có tin nhắn chưa đọc hay không, nếu có mà máy khách chưa đọc, máy chủ gửi tin nhắn đó cho máy khách, tương tự khi máy khách nhận tin nhắn, máy khách thông báo cho máy chủ biết đã đọc. Như vậy có thể đảm bảo mỗi tin nhắn đều được máy cửa hàng nhận được. Tất nhiên, người phát triển cũng có thể sử dụng logic xác nhận của riêng mình.

## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Tự động gọi \Workerman\Protocols\Websocket::encode để đóng gói dữ liệu giao thức websocket trước khi gửi
    $connection->send("hello\n");
};
// Chạy worker
Worker::runAll();
```
