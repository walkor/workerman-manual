# เมธอด __construct
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
สร้างอ็อบเจ็กต์เชื่อมต่อแบบ udp

AsyncUdpConnection ช่วยให้ Workerman เป็นที่รับส่งข้อมูลแบบ udp ในฐานะเซิร์ฟเวอร์

## พารามิเตอร์
พารามิเตอร์: ``` remote_address ```

ที่อยู่ของการเชื่อมต่อ เช่น
``` udp://192.168.1.1:1234 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```



## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // หลังจาก 1 วินาที เปิดใช้งาน UDP ไคลเอนต์เพื่อเชื่อมต่อที่พอร์ต 1234 และส่งข้อความ hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // รับข้อมูลที่เซิร์ฟเวอร์ตอบกลับมา hello
            echo "recv $data\r\n";
            // ปิดการเชื่อมต่อ
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // รับข้อมูลจากไคลเอนต์ AsyncUdpConnection และตอบกลับด้วยข้อความ hello
    $connection->send("hello");
};
Worker::runAll();             
```

หลังจากการทำงาน จะแสดงผลเช่น:
```shell
recv hello
```
