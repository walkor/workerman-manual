# รีไยยูสพอร์ต
> **หมายเหตุ**
> จำเป็นต้องใช้ workerman >= 3.2.1 PHP >= 7.0 ระบบปฏิบัติการ Windows และ Mac OS ไม่รองรับคุณสมบัตินี้

## คำอธิบาย:

```php
bool Worker::$reusePort
```

ตั้งค่าให้ worker ปัจจุบันเปิดใช้งานการ reuse พอร์ตของการฟังก์ชั่น (SO_REUSEPORT) 

การเปิดใช้งานการ reuse พอร์ตทำให้หลายๆ กระบวนการที่ไม่มีความสัมพันธ์สามารถฟังก์ชั่นการเปิดพอร์ตเดียวกันและให้ระบบหลักตัดสินในการทำโหลดแบลนซ์ว่าจะส่งการเชื่อมต่อแบบ socket ให้กับกระบวนการใด ๆ เพื่อการประมวลผล เพื่อหลีกเลี่ยงออกการกระตุ้นกลุ่ม และสามารถเพิ่มประสิทธิภาพของการใช้งานแอปพลิเคชันหลายกระบวนการแบบทำการเชื่อมต่อสั้น

**หมายเหตุ:** คุณสมบัตินี้ต้องใช้ PHP เวอร์ชัน >= 7.0

## ตัวอย่างที่ 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// รัน worker
Worker::runAll();
```

## ตัวอย่างที่ 2: การฟังก์ชั่นการฟังก์ชั่นพอร์ตหลายรายการ (โพรโตคอลหลายรายการ) ของ workerman
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// เมื่อแต่ละกระบวนการเริ่มต้นหลังจากนั้นในกระบวนการปัจจุบันเพิ่มการฟังก์ชั่นการฟังก์ชั่นฟังก์ชั่น
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * หลายกระบวนการมีการฟังก์ชั่นการฟังก์ชั่นนามธรรมเดียวกัน (การฟังก์ชั่นฟังก์ชั่นการใช้พอร์ตไม่ได้ทำการสืบทอดตั้งแต่กระบวนการแม่)
     * ควรเปิดการใช้พอร์ตใหม่เพื่อหลีกเลี่ยงข้อผิดพลาด Address already in use error
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // ทำการฟังก์ชั่นการฟังก์ชั่นการฟังก์ชั่น
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// รัน worker
Worker::runAll();
```
