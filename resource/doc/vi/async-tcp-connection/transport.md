# Thuộc tính vận chuyển

```Yêu cầu (workerman >= 3.3.4)```

Thiết lập thuộc tính vận chuyển, giá trị có thể là [tcp](https://baike.baidu.com/subview/32754/8048820.htm) và [ssl](https://baike.baidu.com/view/525499.htm), mặc định là tcp.

Khi thuộc tính vận chuyển là [ssl](https://baike.baidu.com/view/525499.htm), yêu cầu PHP phải cài đặt [openssl extension](https://php.net/manual/zh/book.openssl.php).

Khi sử dụng Workerman như một máy khách để thiết lập kết nối mã hóa ssl từ máy chủ (kết nối https, kết nối wss, v.v.), vui lòng thiết lập tùy chọn này thành ```ssl```, ví dụ sau đây.

### Ví dụ (kết nối https)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Khi quá trình khởi động bắt đầu, thiết lập một kết nối đến www.baidu.com và gửi dữ liệu để nhận dữ liệu
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // Thiết lập kết nối mã hóa ssl
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Kết nối thành công\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "Kết nối đã đóng\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Mã lỗi:$code Thông báo:$msg\n";
    };
    $connection_to_baidu->connect();
};

// Khởi chạy worker
Worker::runAll();
```
