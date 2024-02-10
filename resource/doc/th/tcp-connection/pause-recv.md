# pauseRecv
## คำอธิบาย:
```php
void Connection::pauseRecv(void)
```

ทำให้การเชื่อมต่อปัจจุบันหยุดการรับข้อมูล โดย callback onMessage ของการเชื่อมต่อนี้จะไม่ถูกเรียก วิธีนี้มีประโยชน์อย่างมากสำหรับควบคุมการไหลของข้อมูลที่ถูกอัปโหลด

## พารามิเตอร์

ไม่มีพารามิเตอร์

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // เพิ่มคุณสมบัติไปยังอ็อบเจกต์ connection เพื่อเก็บจำนวนคำขอที่ถูกส่งมาในการเชื่อมต่อปัจจุบัน
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // หยุดการรับข้อมูลเมื่อการเชื่อมต่อได้รับคำขอไปแล้ว 100 ครั้ง
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// รัน worker
Worker::runAll();
```

## ดูเพิ่มเติม
void Connection::resumeRecv(void) ทำให้อ็อบเจกต์การเชื่อมต่อนั้นๆ กลับมารับข้อมูลอีกครั้ง
