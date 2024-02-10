```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
การเรียกใช้ฟังก์ชั่นหรือเมธอดของคลาสต่าง ๆ ในช่วงเวลาที่กำหนดไว้

หมายเหตุ: ตัวจับเวลาทำงานในกระบวนการปัจจุบัน โดยไม่มีการสร้างกระบวนการหรือเธรดใหม่ใน workerman

### พารามิเตอร์
 ``` time_interval ``` 

ระยะเวลาที่ฟังก์ชันจะทำงาน หน่วยเป็นวินาที รองรับทศนิยม และสามารถแม่นยำถึง 0.001 คือประมาณหนึ่งวินาที

 ``` callback ``` 

ฟังก์ชัน call back `โปรดทราบ: หากฟังก์ชันของ CallBack เป็นเมธอดของคลาส แม้ว่าว่าเมธอด


 ``` args ``` 

พารามิเตอร์ของฟังก์ชัน call back ต้องเป็นอาร์เรย์ที่มีสมาชิกเป็นค่าพารามิเตอร์

 ``` persistent ``` 

เป็นการบ่งบอกว่ามันมีความยั่งยืนหรือไม่ หากต้องการให้ตัวจับเวลาทำงานเพียงครั้งเดียว ให้ระบุเป็น false( งานเพียงครั้งที่กระบวนการที่ครบก็จะขยายตัวเองออกไปโดยอัตโณมให้ต้องไปเรียกใช้`Timer::del()`  ให้ทำการยุติการทำงานโดยอัตโนมัติ )  ค่าเริ่มต้นคือ true ซึ่งหมายถึงการทำงานต่อเนื่อง

### ค่าที่คืนค่า
ส่งค่ากลับให้เป็นจำนวนเต็ม แทน timerid ของตัวจับเวลา สามารถทำการยุติตัวจับเวลาได้โดยการเรียกใช้ `Timer::del($timerid)`

### ตัวอย่าง

#### 1. ฟังก์ชันทำหน้าที่เป็นฟังก์ชันลับ
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

Worker::runAll();
```

### 2. ตั้งจับเวลาระห่่ลบที่เฉพาะตัว
มี worker อยู่ 4 ตัว ตั้งค่าไว้ใน 0 บนตัวจับเวลาไว้
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 worker process, set the timer only in process number 0\n";
        });
    }
};

Worker::runAll();
```
