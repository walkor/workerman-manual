```php
โปรดใช้เมทอดนี้เพื่อเชื่อมต่อแบบไม่ซิงโครนัส
```

### พารามิเตอร์
ไม่มีพารามิเตอร์

### ค่าที่คืน
ไม่มีค่าที่คืน

### ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // เริ่มต้นแค่ตั้งเวลา 1 วินาทีแล้วส่งข้อมูล hi ผ่าน udp client ไปที่พอร์ต 1234
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // ได้รับข้อมูล hello จากเซิร์ฟเวอร์
            echo "recv $data\r\n";
            // ปิดการเชื่อมต่อ
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // ได้รับข้อมูลจาก AsyncUdpConnection และส่งข้อมูล hello กลับไป
    $connection->send("hello");
};
Worker::runAll();    
```

หลังจากการดำเนินการ ผลลัพธ์ที่ปรากฏคือ:
``` 
recv hello
```
