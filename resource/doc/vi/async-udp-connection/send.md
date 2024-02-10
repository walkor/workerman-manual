# Phương thức send
```php
void AsyncUdpConnection::send(string $data)
```
Thực hiện kết nối bất đồng bộ. Phương thức này sẽ trả về ngay lập tức.

### Tham số
 ``` $data ```
 Dữ liệu gửi đến server, dung lượng dữ liệu không vượt quá 65507 byte (dung lượng tối đa của gói tin UDP là 65507 byte). Nếu vượt quá sẽ gửi không thành công.

### Giá trị trả về
Không có giá trị trả về.

### Ví dụ

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 giây sau đó kích hoạt một client udp, kết nối vào cổng 1234 và gửi chuỗi hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Nhận dữ liệu trả về từ server là hello
            echo "Nhận $data\r\n";
            // Đóng kết nối
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Nhận dữ liệu gửi từ client AsyncUdpConnection và trả về chuỗi hello
    $connection->send("hello");
};
Worker::runAll();             
```

Sau khi chạy, in kết quả tương tự:
```shell
Nhận hello
