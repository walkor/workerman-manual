# Phương thức connect

```php
void AsyncUdpConnection::connect()
```

Thực hiện hoạt động kết nối bất đồng bộ. Phương thức này sẽ trả về ngay lập tức.

### Tham số
Không có tham số

### Giá trị trả về
Không có giá trị trả về

### Ví dụ

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Sau 1 giây, khởi động một kết nối udp, kết nối cổng 1234 và gửi chuỗi hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Nhận dữ liệu trả về từ máy chủ là hello
            echo "recv $data\r\n";
            // Đóng kết nối
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Nhận dữ liệu từ khách hàng AsyncUdpConnection, trả về chuỗi hello
    $connection->send("hello");
};
Worker::runAll();
```

Sau khi thực hiện, in ra tương tự như sau:
```
recv hello
```
