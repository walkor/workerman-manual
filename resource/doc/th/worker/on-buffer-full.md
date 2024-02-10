# onBufferFull
## คำอธิบาย:
```php
callback Worker::$onBufferFull
```

ทุกการเชื่อมต่อจะมี buffer สำหรับการส่งที่ระดับแอปพลิเคชันเป็นลมอยู่ ถ้าความเร็วในการรับของผู้ใช้น้อยกว่าความเร็วในการส่งของเซิร์ฟเวอร์ ข้อมูลจะถูกเก็บไว้ใน buffer ของระดับแอปพลิเคชัน ถ้า buffer เต็มแล้วจะเรียกใช้ callback onBufferFull

buffer ส่งขนาดใหญ่[TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)มีค่าเริ่มต้นที่ 1MB สามารถตั้งค่าขนาด buffer ได้แบบไดนามิกสำหรับการเชื่อมต่อปัจจุบัน เช่น:
```php
// ตั้งค่าขนาด buffer ส่งของการเชื่อมต่อปัจจุบัน หน่วยเป็นไบต์
$connection->maxSendBufferSize = 102400;
```
และยังสามารถใช้[TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md)เพื่อตั้งค่าขนาด buffer เริ่มต้นสำหรับการเชื่อมต่อทั้งหมด เช่นโค้ด:
```php
use Workerman\Connection\TcpConnection;
// ตั้งค่าขนาด buffer ส่งของการเชื่อมต่อทั้งหมดเป็นค่าเริ่มต้น หน่วยเป็นไบต์
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

callback นี้**อาจ**ถูกเรียกทันทีหลังจากการเรียก Connection::send เช่น การส่งข้อมูลขนาดใหญ่หรือทำการส่งข้อมูลไปทับซ้ำโดยรวดเร็วไปยังอีกฝั่ง เนื่องจากเหตุผลต่างๆ ข้อมูลก็ถูกเหลือไว้ใน buffer ส่งของการเชื่อมต่อที่เกี่ยวกับ การเกินข้างบนของ```TcpConnection::$maxSendBufferSize```เวลาจะเรียกใช้

เมื่อเกิดเหตุการณ์ onBufferFull นักพัฒนาควรทำหน้าที่ เช่น หยุดการส่งข้อมูลไปที่อีกฝั่ง รอให้ข้อมูลใน buffer ส่งเสร็จสิ้น (เหตุการณ์ onBufferDrain) เป็นต้น

เมื่อเรียกใช้ Connection::send(`$A`) ครั้งที่ส่งเป็นโอกาสที่เรียกใช้ onBufferFull เหมือนกับข้อมูลที่จะส่งที่จะถูกเก็บใน buffer ส่ง ไม่ว่าจะใหญ่ขนาดเท่าไร แม้แต่มันใหญ่กว่า`TcpConnection::$maxSendBufferSize` ข้อมูลที่จะส่งในครั้งนี้ยังคงถูกเก็บไว้ใน buffer ส่ง นอกเหนือจาก buffer ส่งภายในีจระับไว و܇ณัวการเกินขอบเขต```TcpConnection::$maxSendBufferSize``` จะเรียกใช้ onBufferFull

โดยสรุป รำับว่า ถ้า buffer ส่งยังไม่เต็ม ไมัสำคัญแม้แต่มีแค่พื้นที่ไว้จิ่การเรียกใช้ Connection::send(```$A```) จะเอา```$A```ไปใส่ใน buffer การส่ง หากอย่างไรก็ตามหลังจากเอาไปในbuffer การส่งแล้วถ้าขนาดของ buffer การส่งมีค่ามากกว่า```TcpConnection::$maxSendBufferSize``` การจําเงิบจะเรียกใช้ onBufferFull回。

## พารามิเตอร์ของฟังก์ชัน callback

 ``` $connection ```

วัตถุการเชื่อมต่อ ซึ่งหมายควา  [TcpConnection ที่เป็นอิสปรินึเมน](../tcp-connection.md)  ใช้ในการดําเนินการ เช่น [ส่งข้อมูล](../tcp-connection/send.md) [ปิดการเชื่อมต่อ](../tcp-connection/close.md) เป็นต้น

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
// ทำให้ worker ทำงาน
Worker::runAll();
```

เรายังสามารถ[อ้างอิงที่นี่](../faq/callback_methods.md)ไปยังวิธีการตั้งค่า callback อื่นๆได้

## ดูเพิ่มเติม
onBufferDrain เมื่อข้อมูลใน buffer ส่งของการเชื่ออถอยหมดการส่งทั้งหมดของการเชื่อมต่อ เกิดขึ้น
