# การส่งและรับข้อมูลฐานสิบหก

**การรับข้อมูลฐานสิบหก**

เมื่อได้รับข้อมูลแล้ว สามารถใช้ฟังก์ชั่น `bin2hex($data)` เพื่อแปลงข้อมูลเป็นรหัสฐานสิบหก

**การส่งข้อมูลฐานสิบหก**

ก่อนส่งข้อมูล ใช้ `hex2bin($data)` เพื่อแปลงข้อมูลฐานสิบหกเป็นข้อมูลทวิศิษย์เพื่อส่ง

**ตัวอย่าง:**

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8080');
$worker->onMessage = function(TcpConnection $connection, $data){
    // รับข้อมูลฐานสิบหก
    $hex_data = bin2hex($data);
    // ส่งข้อมูลฐานสิบหกไปยังไคลเอนต์
    $connection->send(hex2bin($hex_data));
};
Worker::runAll();
```
