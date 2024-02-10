# globalEvent

## คำอธิบาย:
```php
static Event Worker::$globalEvent
```

คุณสมบัตินี้เป็นคุณสมบัติสถิตทั่วไปที่เป็นตัวอย่างของ eventloop ที่สามารถลงทะเบียนเหตุการณ์การอ่านหรือเขียนของไฟล์ดีสคริปเตอร์หรือเหตุการณ์สัญญาณได้

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // เมื่อกระบวนการได้รับสัญญาณ SIGALRM ให้แสดงข้อมูลบางอย่าง
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// เริ่มการทำงานของ worker
Worker::runAll();
```

## การทดสอบ
หลังจาก Workerman เริ่มต้น จะแสดงผลของ pid ของกระบวนการปัจจุบัน (ตัวเลขหนึ่ง) ให้รันคำสั่งใน command line ดังนี้
```bash
kill -SIGALRM  pid ของกระบวนการ
```

เซิร์ฟเวอร์จะแสดงผลดังนี้
```bash
Get signal SIGALRM
```
