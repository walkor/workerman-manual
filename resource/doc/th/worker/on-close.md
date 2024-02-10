# onClose
## คำอธิบาย:
```php
callback Worker::$onClose
```

เมื่อการเชื่อมต่อของ Client กับ Workerman ถูกตัดสัญญาณ ฟังก์ชั่น callback นี้จะถูกเรียกใช้ ไม่ว่าการเชื่อมต่อจะถูกตัดสัญญาณไปอย่างไร แม้กระทั่งการตัดสัญญาณก็จะทำให้เกิดการเรียกใช้ ```onClose``` ทุกการเชื่อมต่อจะทำให้เกิดการเรียกใช้ ```onClose``` ครั้งเดียว

โปรดทราบ: ถ้าการเชื่อมต่อไปยัง Workerman ถูกตัดสัญญาณ เช่น โดยเหตุการณ์เฉียบพลันเช่นการตัดสัญญาณอินเทอร์เน็ตหรือการตัดสัญญาณไฟฟ้า Workerman จะไม่สามารถทราบว่าการเชื่อมต่อได้ถูกตัดสัญญาณ ซึ่งทำให้ไม่สามารถเรียกใช้ ```onClose``` ได้ทันที ซึ่งควรจะใช้ heartbeating ในระดับการใช้งานแอปพลิเคชั้นเพื่อแก้ไขปัญหานี้ การทำ heartbeating ใน Workerman สามารถดูรายละเอียดได้จาก[ที่นี่](../faq/heartbeat.md) ถ้าใช้ GatewayWorker framework สามารถใช้กลไก heartbeating ของ GatewayWorker framework โดยตรง ดูรายละเอียดได้จาก[ที่นี่](https://doc2.workerman.net/heartbeat.html)

เนื่องจาก udp ไม่มีการเชื่อมต่อ การใช้ udp จะไม่มีการเรียก callback onConnect และไม่มีการเรียก callback onClose เช่นกัน

## พารามิเตอร์ของฟังก์ชัน callback

 ``` $connection ```

อ็อบเจ็กต์เชื่อมต่อ หรือ [TcpConnection instance](../tcp-connection.md) ใช้ในการจัดการการเชื่อมต่อของ Client เช่นการ[ส่งข้อมูล](../tcp-connection/send.md) หรือ [ปิดการเชื่อมต่อ](../tcp-connection/close.md) เป็นต้น

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// รัน worker
Worker::runAll();
```

หมายเหตุ: นอกจากการใช้ฟังก์ชันแบบไม่มีชื่อเป็น callback แล้ว ยังสามารถ[ดูเพิ่มเติมได้ที่นี่](../faq/callback_methods.md)วิธีการเขียน callback อื่นๆ ได้
