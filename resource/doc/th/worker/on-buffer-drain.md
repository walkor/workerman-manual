# onBufferDrain
## คำอธิบาย:
```php
callback Worker::$onBufferDrain
```

ทุกการเชื่อมต่อจะมีที่เก็บข้อมูลสำหรับการส่งข้อมูลที่แยกออกเป็นชั้น ขนาดของพื้นที่เก็บข้อมูลถูกกำหนดโดย ```TcpConnection::$maxSendBufferSize``` ค่าเริ่มต้นคือ 1 เมกะไบต์ และสามารถกำหนดค่าเพิ่มเติมได้ หลังจากที่กำหนดค่าได้จะมีผลทุกการเชื่อมต่อ

Callback นี้จะทำงานเมื่อข้อมูลในพื้นที่เก็บของการส่งข้อมูลทุกหนเสร็จสมบูรณ์ ซึ่งมักจะใช้ร่วมกับ onBufferFull ตัวอย่างเช่น เมื่อเกิด onBufferFull จะหยุดการส่งข้อมูลไปยังอีกฝักหรือเครื่องต่อ แต่เมื่อเกิด onBufferDrain จะฟื้นฟูการเขียนข้อมูลในพื้นที่เก็บข้อมูลได้อีกครั้ง


## พารามิเตอร์ของฟังก์ชัน Callback
``` $connection ```

อ็อบเจ็กต์การเชื่อมต่อ หรือ [TcpConnection instance](../tcp-connection.md) ที่ใช้สำหรับดำเนินการที่เกี่ยวกับการเชื่อมต่อของผู้ใช้ เช่น [ส่งข้อมูล](../tcp-connection/send.md), [ปิดการเชื่อมต่อ](../tcp-connection/close.md) ฯลฯ


## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "buffer drain and continue send\n";
};
// ปฏิบัติการเวิร์กเกอร์
Worker::runAll();
```

เกรด: นอกเหนือจากการใช้ฟังก์ชันไม่มีชื่อเป็น Callback ยังสามารถ [ดูเพิ่มเติมที่นี่](../faq/callback_methods.md) วิธีการเขียน Callback อื่น ๆ ได้

## ดูเพิ่มเติม
onBufferFull เมื่อพื้นที่เก็บข้อมูลสำหรับการส่งข้อมูลของการเชื่อมต่อแบบแอปพลิเคซเต็ม
