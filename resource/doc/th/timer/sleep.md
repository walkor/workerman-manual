# การหน่วงเวลา

```php
int \Workerman\Timer::sleep(float $delay)
```

คล้ายกับฟังก์ชั่น `sleep()` ที่ภายใน PHP แต่ที่แตกต่างคือ `Timer::sleep()` จะเป็นการไม่บล็อก (non-blocking) (ไม่ทำให้กระบวนการปัจจุบันถูกบล็อก)

> **โปรดทราบ**
> คุณลักษณะนี้ต้องการ workerman>=5.0
> คุณลักษณะนี้ต้องการการติดตั้ง composer require revolt/event-loop ^1.0.0 หรือใช้ Swoole/Swow เป็นตัวขับเคลื่อนเหตุการณ์

### พารามิเตอร์
 ``` delay ``` 

เวลาที่ต้องการจะหน่วงเป็นเวลาสองหรือนาที โดยหน่วงเป็นหน่วยวินาที และรองรับทศนิยม สามารถระบุได้ถึง 0.001 ซึ่งหมายถึงถึงระดับมิลลิวินาที

### ค่าคืน
ไม่มีค่าที่ถูกคืน

### ตัวอย่าง

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // หน่วงเวลา 1.5 วินาที แล้วส่งข้อมูล
    Timer::sleep(1.5);
    // ส่งข้อมูล
    $connection->send('hello workerman');
};

Worker::runAll();
```
