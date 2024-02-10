```php
<?php
boolean \Workerman\Timer::del(int $timer_id)
```
ลบตัวตั้งเวลาที่กำหนด

### พารามิเตอร์
``` timer_id ```

id ของตัวตั้งเวลา คือจำนวนเต็มที่คืนจากตัวฟังก์ชัน add

### ค่าคืน
boolean


### ตัวอย่าง
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// เปิดการทำงานทรัพยากรหลายๆ ทรัพยากรสำหรับการทำงานแบบตัวตั้งเวลา กรุณาใช้หลายๆ กระบวนการพร้อมกันอาจจะเกิดความไม่สอดคล้อง
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // ทำงานทุกๆ 2 วินาที
    $timer_id = Timer::add(2, function()
    {
        echo "ทำงาน\n";
    });
    // 20 วินาที่ต่อมามีงานที่เกิดขึ้นแบบครั้งเดียว ลบตัวตั้งเวลาที่เกิดขึ้นทุก 2 วินาที่
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// ประยุกต์ใช้ worker
Worker::runAll();
```

### ตัวอย่าง(การหลุดตัวตั้้งเวลาในการเรียกกลับของตัวตั้งเวลาปัจจุบัน)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // โปรดทราบ การใช้ id ตัวตั้งเวลาปัจจุบันจะต้องใช้การอ้างอิงรูปแบบ(&)
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // ลบตัวตั้งเวลาหลังจากทำงาน 10 ครั้ง
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// ประยุกต์ใช้ worker
Worker::runAll();
```
