```php
public function close($data = '')
```

ปิดการเชื่อมต่ออย่างปลอดภัยและเรียกใช้งาน callback ของการปิดการเชื่อมต่อ (`onClose`) โดยเฉพาะ

แม้ว่า udp จะเป็นการเชื่อมต่อแบบไม่มีการเชื่อมต่อ แต่อ็อบเจ็กต์ของ AsyncUdpConnection ที่เกี่ยวข้องจะยังคงอยู่ในหน่วยความจำตลอดเวลา จึงจำเป็นต้องเรียกใช้เมธอด close เพื่อที่จะปล่อยอ็อบเจ็กต์การเชื่อมต่อ udp ที่เกี่ยวข้องออกจากหน่วยความจำ มิฉะนั้นอ็อบเจ็กต์การเชื่อมต่อ udp จะยังคงอยู่ในหน่วยความจำทำให้เกิดการรั่วหน่วยความจำ

## พารามิเตอร์

 ``` $data ```

พารามิเตอร์ที่ไม่จำเป็น ข้อมูลที่ต้องการส่ง (หากมีการระบุโปรโตคอล จะถูกเรียกใช้เมธอด encode ของโปรโตคอลเพื่อแพ็คข้อมูล ```$data```) เมื่อส่งข้อมูลเสร็จหรือหมดระยะเวลาการส่ง การเชื่อมต่อจะถูกปิด ซึ่งทำให้เรียกใช้ callback ของการปิดการเชื่อมต่อได้

ขนาดของข้อมูลต้องไม่เกิน 65507 ไบต์ มิฉะนั้นจะส่งล้มเสีย

### ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 วินาทีหลังจากนั้นเริ่มต้น udp ไคลเอ็นต์ ที่เชื่อมต่อไปยังพอร์ต 1234 และส่งสตริง hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // รับข้อมูลที่เซิร์ฟเวอร์ตอบกลับมา hello
            echo "recv $data\r\n";
            // ปิดการเชื่อมต่อ
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // รับข้อมูลที่ถูกส่งจากไคลเอ็นต์ AsyncUdpConnection และตอบกลับด้วยสตริง hello
    $connection->send("hello");
};
Worker::runAll();             
```

เมื่อทำฉลดผลลัพธ์เป็นเช่น
````
recv hello
````
