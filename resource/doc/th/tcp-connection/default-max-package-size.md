# maxPackageSize

## คำอธิบาย:
```php
static int Connection::$defaultMaxPackageSize
```

คุณสมบัตินี้เป็นคุณสมบัติสถิติระดับโลก ที่ใช้ในการตั้งค่าขนาดสูงสุดของแพคเกจที่แต่ละการเชื่อมต่อสามารถรับได้ หากไม่ได้ตั้งค่า ค่าเริ่มต้นคือ 10 เมกะไบต์

หากข้อมูลที่ถูกส่งมามีขนาดของแพคเกจที่ถูกแยกวิเคราะห์ (ค่าที่คืนจาก input ของคลาสโปรโตคอล) มากกว่า ```Connection::$defaultMaxPackageSize``` จะถือว่าเป็นข้อมูลที่ผิดกฎ และการเชื่อมต่อจะถูกตัดสินบ.

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ตั้งค่าขนาดสูงสุดของแพคเกจที่แต่ละการเชื่อมต่อสามารถรับได้เป็น 1024000 ไบต์
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// รันเซิร์ฟเวอร์
Worker::runAll();
```
