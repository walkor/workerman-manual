การ Debugging พื้นฐาน

Workerman มีสองโหมดการทำงาน คือ โหมด Debugging และ โหมดการทำงานแบบ daemon

เข้าสู่โหมด Debugging โดยการรัน ```php start.php start``` ในโหมดนี้ การพิมพ์ของฟังก์ชันเช่น ```echo、var_dump、var_export``` ในโค้ดจะแสดงผลบนทีนเทอร์มินั่นหมายความว่าเมื่อปิดทีนเทอร์มินัระบบ Workerman ก็จะทำการสิ้นสุดการทำงานทั้งหมด

ส่วนการรัน ```php start.php start -d``` จะเข้าสู่โหมด daemon ซึ่งเป็นโหมดการทำงานที่ใช้งานจริงบนเว็บไซต์ โดยการปิดทีนเทอร์มินัไม่มีผลต่อการทำงาน

หากต้องการให้สามารถดูการพิมพ์ของฟังก์ชันเช่น ```echo、var_dump、var_export``` ในโหมดการทำงานแบบ daemon สามารถตั้งค่า Worker::$stdoutFile ได้ เช่น

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ส่งผลการพิมพ์ไปยังไฟล์ที่ Worker::$stdoutFile ระบุ
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```
โดยฉะนั้น การพิมพ์ของฟังก์ชันทั้งหมดเช่น ```echo、var_dump、var_export``` จะถูกเขียนลงไปในไฟล์ที่ระบุใน ```Worker::$stdoutFile``` โปรดทราบว่าที่แขนงของ ```Worker::$stdoutFile``` จะต้องมีสิทธิ์ในการเขียนไฟล์ได้
