# ส่ง
## คำอธิบาย:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

ส่งข้อมูลไปยังไคลเอ็นต์

## พารามิเตอร์

``` $data ```

ข้อมูลที่ต้องการส่ง หากพารามิเตอร์นี้ถูกกำหนดเมื่อกำหนดคลาส Worker ก็จะเรียกใช้เมทอด encode ของโปรโตคอลเพื่อทำการแพ็คข้อมูลโปรโตคอลและส่งไปยังไคลเอ็นต์

``` $raw ```

บอกว่าจะส่งข้อมูลแบบสแตรมหรือไม่ ซึ่งหมายถึงไม่เรียกใช้เมทอด encode ของโปรโตคอล ค่าเริ่มต้นคือ false หมายความว่าจะเรียกใช้เมทอด encode ของโปรโตคอลอัตโนมัติ

## ค่าการส่งคืน

true หมายถึงข้อมูลถูกเขียนไปยังเครื่องเชื่อมต่อของการทำงานของเซิร์ฟเวอร์สำเร็จ

null หมายถึงข้อมูลถูกเขียนไปยังเครื่องเชื่อมต่อของการทำงานของไคลเอ็นต์แล้ว โดยรอการเขียนลงในบัฟเฟอร์การส่งของการทำงานของระบบ

false หมายถึงการส่งล้มเหลว สาเหตุเกิดจากการเชื่อมต่อของไคลเอ็นต์ถูกปิด หรือบัฟเฟอร์การส่งของการทำงานของเครื่องเชื่อมต่อนี้เต็ม

## หมายเหตุ
การส่งค่าคืน ```true``` เพียงแค่บอกว่าข้อมูลได้ถูกเขียนลงในบัฟเฟอร์การส่งของการทำงานของเซิร์ฟเวอร์เท่านั้น และไม่ได้หมายความว่าข้อมูลถูกส่งไปยังบัฟเฟอร์รับของเซิร์ฟเวอร์ตรงข้ามแล้ว บวกับการที่มีการติดต่อของไคลเอ็นต์อย่างปกติ ข้อมูลเท่าที่เกือบสามารถถือว่าสามารถถูกส่งไปยังอีกฝั่งได้ 100% ได้

เนื่องจากข้อมูลบัฟเฟอร์การส่งของเครื่องเชื่อมต่อจะถูกส่งอย่างไม่สม่นสมาตาต่อเครื่องเชื่อมต่ออีกฝั่งตามแบบไม่ด่วน ระบบปฏิบัติการไม่มีการยืนยันการบันทึกข้อมูลเชิงดุุน ทำให้ทำงานของระบบชั้นประยุกต์ไม่สามารถทราบได้ว่าข้อมูลบัฟเฟอร์ส่งของการทำงานได้เริ่มการส่งแล้วหรือยัง, รวมถึงไม่สามารถทราบได้ว่าการส่งบัฟเฟอร์การส่งของการทำงานของเครื่องเชื่อมต่อนั้นสามารถส่งข้อมูลได้สำเร็จหรือไม่ จึงเป็นเหตุผลสำคัญทำให้ workerman ไม่สามารถถอดรหัสการยืนยันข้อความได้อย่างตรงไปตรงมา

หากงานธุรกิจต้องการยืนยันว่าผลิตภัณฑ์ได้ถึงทุกฝั่ง นั้นสามารถเพิ่มกลไกยืนยันได้ ซึ่งคล้ายกันตามเงื่อนไขของธุรกิจก็มีกลไกยืนยันที่หลากหลายเช่นกัน

ตัวอย่างเช่นระบบการสนทนาอาจใช้กลไกยืนยันแบบนี้ ให้ทุกข้อความถูกเก็บไว้ในฐานข้อมูล และแต่ละข้อความมีฟิลด์สำหรับบอกว่าหมดอายุหรือยัง การที่ไคลเอ็นต์รับข้อความแล้วจะส่งกลับไปยังเซิร์ฟเวอร์ได้ แล้วเซิร์ฟเวอร์จะเปลี่ยนสถานะลงในฐานข้อมูลว่าไคลเอ็นต์ได้รับข้อความแล้ว และเมื่อไคลเอ็นต์ติดต่อเซิร์ฟเวอร์เพื่อดาวน์โหลดหรือต่อการเชื่อมต่อ ฐานข้อมูลก็จะเช็คว่ายังมีข้อความที่ยังไม่รับไว้หรือไม่ ถ้ามีก็จะส่งไปยังไคลเอ็นต์ และไคลเอ็นต์ได้รับข้อความก็จะส่งข้อมูลกลับไปให้เซิร์ฟเวอร์ว่าได้รับข้อความแล้ว นอกจากนี้ทางผู้พัฒนาระบบก็สามารถใช้กลไกยืนยันของตนเองได้และอีกหลายวิธีของกลไกยืนยันที่หลากหลายให้เลือกใช้ได้

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // จะเรียกใช้เมทอด \Workerman\Protocols\Websocket::encode ด้วยเพื่อทำการแพ็คข้อมูลเป็นโปรโตคอลของเว็บโฮสต์ แล้วนำไปส่ง
    $connection->send("hello\n");
};
// เริ่มการทำงานของเซิร์ฟเวอร์
Worker::runAll();
```