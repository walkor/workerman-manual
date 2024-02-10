# onWorkerStart
## คำอธิบาย:
```php
callback Worker::$onWorkerStart
```

กำหนดฟังก์ชัน callback สำหรับเมื่อกระบวนการย่อยของ Worker เริ่มทำงาน โดยฟังก์ชันนี้จะถูกเรียกทุกครั้งที่กระบวนการย่อยเริ่มทำงาน

โปรดทราบ: onWorkerStart จะถูกเรียกใช้เมื่อกระบวนการย่อยเริ่มทำงาน ถ้ามีการเปิดกระบวนการย่อยหลายตัว(```$worker->count > 1```) ฟังก์ชันนี้จะถูกเรียกใช้ ```$worker->count``` ครั้งทั้งหมด

## พารามิเตอร์ของฟังก์ชัน callback

 ``` $worker ```

คืออ็อบเจ็กต์ของ Worker


## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Worker starting...\n";
};
// เริ่มต้น Worker
Worker::runAll();
```

เกริ่นสิ่งที่สำคัญ: สำหรับการใช้ฟังก์ชัน callback อีกทางหนึ่ง สามารถ[ดูตัวอย่างได้ที่นี่](../faq/callback_methods.md)
