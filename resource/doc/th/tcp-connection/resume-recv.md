# resumeRecv
## คำอธิบาย:
```php
void Connection::resumeRecv(void)
```

ทำให้การเชื่อมต่อปัจจุบันสามารถดำเนินการรับข้อมูลต่อไปได้ วิธีการนี้จะมีประโยชน์อย่างมากสำหรับการควบคุมการไหลของการถ่ายโอนข้อมูล

## พารามิเตอร์

ไม่มีพารามิเตอร์

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // เพิ่มคุณลักษณะไปที่วัตถุ connection เพื่อเก็บจำนวนคำขอที่ถูกส่งมาในการเชื่อมต่อปัจจุบัน
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // หยุดการรับข้อมูลหลังจากได้รับคำขอ 100 คำขอ
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // 30 วินาทีทีถัดมาจะทำการดำเนินการรับข้อมูลอีกครั้ง
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// เริ่มการทำงานของ worker
Worker::runAll();
```

## ดูเพิ่มเติม
void Connection::pauseRecv(void) ทำให้วัตถุการเชื่อมต่อหยุดการรับข้อมูลที่เกี่ยวข้อง
