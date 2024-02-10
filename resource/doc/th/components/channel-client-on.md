# ที่ on
**``` (ต้องการ Workerman เวอร์ชัน >= 3.3.0) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
สมัครสมาชิกในเหตุการณ์ ```$event_name``` และลงทะเบียนฟังก์ชัน callback ที่เกิดขึ้นในช่วงเวลา ```$callback_function```

## พารามิเตอร์ของฟังก์ชัน callback

 ``` $event_name ```

ชื่อเหตุการณ์ที่สมัครสมาชิกไว้, มีค่าอะไรก็ได้

 ``` $callback_function ```

ฟังก์ชัน callback ที่เกิดขึ้นเมื่อเหตุการณ์เกิดขึ้น ลักษณะของฟังก์ชันคือ ```callback_function(mixed $event_data)``` ในนั้นมี ```$event_data``` ที่เป็นข้อมูลเหตุการณ์ที่ถูกตีพิมพ์ (publish) ขึ้นมา

โปรดทราบ:

หากลงทะเบียนฟังก์ชัน callback สองฟังก์ชัน หรือมากกว่านั้น ฟังก์ชัน callback ที่หลังจะแทนที่ฟังก์ชัน callback ก่อนหน้านั้น


## ตัวอย่าง
Worker แบบ multi-process (หลายเซิร์ฟเวอร์) โฮสต์หนึ่งคนส่งข้อความไปยังทุกคน

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// กำหนดเซิร์ฟเวอร์ Channel
$channel_server = new Channel\Server('0.0.0.0', 2206);

// WebSocket เซิร์ฟเวอร์
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// เมื่อโปรเซสของ worker เริ่ม
$worker->onWorkerStart = function($worker)
{
    // ลูกค้าช่องเชื่อมต่อไปยังเซิร์ฟเวอร์ Channel
    Channel\Client::connect('127.0.0.1', 2206);
    // สมัครสมาชิกเหตุการณ์ broadcast และลงทะเบียนฟังก์ชัน callback ของเหตุการณ์
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // ส่งข้อความไปยังทุกโครงการของ worker ปัจจุบัน
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // ให้ข้อมูลที่มาจากลูกค้าเป็นข้อมูลเหตุการณ์
   $event_data = $data;
   // ตีพิมพ์เหตุการณ์ broadcast ไปยังโปรเซสทุกๆ worker
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```


**การทดสอบ**

เปิดเบราว์เซอร์ Chrome และกด F12 เพื่อเปิดคอนโซล debugger ในส่วนของ Console พิมพ์เข้าไป (หรือวางโค้ดต่อไปนี้ลงใน html แล้วรันด้วย JS)

การเชื่อมต่อที่รับข้อมูล
```javascript
// 127.0.0.1 ให้เปลี่ยนเป็น IP ที่ Workerman อยู่จริง
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("ได้รับข้อความจากเซิร์ฟเวอร์：" + e.data);
};
```


การประกาศข้อความ
```javascript
ws.send('สวัสดีชาวโลก');
```
