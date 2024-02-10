# วิธีการใช้ send

```php
void AsyncUdpConnection::send(string $data)
```
ดำเนินการเชื่อมต่อแบบไม่ซิงโครนัส วิธีนี้จะทำการทำงานและคืนค่าทันที

### พารามิเตอร์
 ``` $data ```
ข้อมูลที่จะส่งไปยังเซิร์ฟเวอร์ ขนาดข้อมูลต้องไม่เกิน 65507 ไบต์ (ขนาดการส่งข้อมูล UDP ต่อหนึ่งแพ็คสูงสุดคือ 65507 ไบต์) มิฉะนั้นจะส่งไม่สำเร็จ

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
    // 1 วินาทีหลังจากนั้นเริ่มต้นแคลเลียนต์ UDP ที่เชื่อมต่อไปยังพอร์ต 1234 และส่งข้อความ hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // ได้รับข้อมูลที่เซิร์ฟเวอร์ตอบกลับมา hello
            echo "recv $data\r\n";
            // ปิดการเชื่อมต่อ
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // ได้รับข้อมูลจากลูกค้า AsyncUdpConnection และส่งข้อความกลับว่า hello
    $connection->send("hello");
};
Worker::runAll();             
```

เมื่อทำการดำเนินการแล้ว จะพิมพ์เช่นนี้:
```
recv hello
```
