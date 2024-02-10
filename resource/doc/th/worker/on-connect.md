# onConnect
## คำอธิบาย:
```php
callback Worker::$onConnect
```

เมื่อเชื่อมต่อระหว่าง client กับ Workerman เสร็จสมบูรณ์ (หลังจากทำ TCP 3-way handshake) จะเกิดการเรียกใช้ callback function นี้ โดยทุกครั้งที่มีการเชื่อมต่อ จะมีการเรียกใช้ callback ```onConnect``` ครั้งเดียวเท่านั้น

โปรดทราบ: เหตุการณ์ `onConnect` อย่างเดียวจะแสดงถึงการเชื่อมต่อระหว่าง client กับ Workerman ทำ 3-way handshake สำเร็จเท่านั้น ในขณะนี้ client ยังไม่ได้ส่งข้อมูลอะไรมา แต่นอกจากการใช้ ```$connection->getRemoteIp()``` เพื่อได้ IP ของฝั่งตรงข้ามแล้ว ไม่มีข้อมูลหรือข้อมูลที่สามารถใช้ในการระบุว่าฝักตรงข้ามคือใคร ดังนั้นในเหตุการณ์ onConnect จึงไม่สามารถระบุตรวจสอบว่าฝั่งตรงข้ามคือใคร หากต้องการทราบว่าฝั่งตรงข้ามคือใคร จะต้องมีการส่งข้อมูลการรับรองตัวตน โดยอาจจะเป็น token หรือชื่อผู้ใช้และรหัสผ่านและอื่น ๆ ใน[Callback onMessage](on-message.md) 

เนื่องจาก UDP เป็นโปรโตคอลที่ไม่มีการเชื่อมต่อ ดังนั้นเมื่อใช้ UDP จะไม่เกิดการเรียกใช้ callback onConnect และไม่เกิดการเรียกใช้ callback onClose

## พารามิเตอร์ของฟังก์ชัน callback

 ``` $connection ```

 Object ของการเชื่อมต่อ หรือแบบ TcpConnection instance ที่ใช้ในการจัดการการเชื่อมต่อกับ client เช่น [ส่งข้อมูล](../tcp-connection/send.md) หรือ [ปิดการเชื่อมต่อ](../tcp-connection/close.md) และอื่น ๆ

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// รัน worker ทั้งหมด
Worker::runAll();
```

เรียนร้อย: นอกจากการใช้ฟังก์ชันไม่มีชื่อเป็น callback ยังสามารถ [ดูตัวอย่างได้ที่นี่](../faq/callback_methods.md) ใช้วิธีการเขียน callback อื่น ๆ ได้
