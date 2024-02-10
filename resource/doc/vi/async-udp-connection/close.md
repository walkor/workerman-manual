```php
void Connection::close(mixed $data = '')
```

Đóng kết nối an toàn và gọi lại ```onClose``` callback của kết nối.

Mặc dù UDP không có kết nối, nhưng đối tượng AsyncUdpConnection tương ứng vẫn được giữ trong bộ nhớ, và phải gọi phương thức close để giải phóng đối tượng kết nối UDP tương ứng, nếu không đối tượng kết nối UDP này sẽ tiếp tục tồn tại trong bộ nhớ, gây ra rò rỉ bộ nhớ.

## Tham số

 ``` $data ```

Tham số tùy chọn, dữ liệu cần gửi đi (nếu có đặc tả giao thức, thì dữ liệu ```$data``` sẽ tự động gọi phương thức encode của giao thức để đóng gói dữ liệu ```$data```), sau khi dữ liệu được gửi đi, kết nối sẽ được đóng và sau đó sẽ gọi lại callback onClose.

Kích thước dữ liệu không được vượt quá 65507 byte, nếu vượt quá sẽ gửi không thành công.

### Ví dụ 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 giây sau khởi chạy một client UDP, kết nối vào cổng 1234 và gửi chuỗi "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Nhận dữ liệu trả về từ máy chủ là "hello"
            echo "recv $data\r\n";
            // Đóng kết nối
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Nhận dữ liệu từ client AsyncUdpConnection, trả về chuỗi "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

Sau khi chạy, sẽ có in tương tự như sau:
```php
recv hello
```
