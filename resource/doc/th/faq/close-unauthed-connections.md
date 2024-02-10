# ปิดการเชื่อมต่อที่ไม่ได้รับการรับรอง

**คำถาม:** 

วิธีการปิดการเชื่อมต่อของลูกค้าที่ไม่ได้ส่งข้อมูลเป็นเวลาที่กำหนด, 
เช่น ปิดการเชื่อมต่อของลูกค้าที่ไม่ได้ส่งข้อมูลเป็นเวลา 30 วินาที, 
เพื่อให้การเชื่อมต่อที่ไม่ได้รับการรับรองต้องรับรองตัวตรวจไว้ภายในเวลาที่กำหนด

**คำตอบ:**

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('xxx://x.x.x.x:x');
$worker->onConnect = function(TcpConnection $connection)
{
    // เพิ่มคุณสมบัติ auth_timer_id ลงในอ็อบเจ็ค $connection เป็นชั่วคราว
    // ตั้งเวลาปิดการเชื่อมต่อใน 30 วินาที  หากไม่มีการส่งการตรวจสอบภายใน 30 วินาที
    $connection->auth_timer_id = Timer::add(30, function()use($connection){
        $connection->close();
    }, null, false);
};
$worker->onMessage = function(TcpConnection $connection, $msg)
{
    $msg = json_decode($msg, true);
    switch($msg['type'])
    {
    case 'login':
        ... ข้าม
        // การตรวจสอบสำเร็จ ลบตัวนับเวลาเพื่อป้องกันการปิดการเชื่อมต่อ
        Timer::del($connection->auth_timer_id);
        break;
         ... ข้าม
    }
    ... ข้าม
}
```
