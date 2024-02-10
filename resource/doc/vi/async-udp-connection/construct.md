# Phương thức __construct
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Tạo một đối tượng kết nối UDP.

AsyncUdpConnection cho phép Workerman hoạt động như một máy khách kết nối dữ liệu UDP với máy chủ từ xa.

## Tham số
Tham số: `remote_address`

Địa chỉ kết nối, ví dụ
 ``` udp://192.168.1.1:1234 ```
 ``` frame://192.168.1.1:8080 ```
 ``` text://192.168.1.1:8080 ```


## Ví dụ

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 giây sau đó khởi động một máy khách UDP, kết nối vào cổng 1234 và gửi chuỗi hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Nhận dữ liệu phản hồi từ máy chủ là hello
            echo "recv $data\r\n";
            // Đóng kết nối
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // Nhận dữ liệu từ máy khách AsyncUdpConnection, trả về chuỗi hello
    $connection->send("hello");
};
Worker::runAll();             
```

Sau khi thực hiện, in tương tự như sau:
```
recv hello
```
