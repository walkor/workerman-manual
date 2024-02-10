# สามารถรีโหลด
## คำอธิบาย:
```php
bool Worker::$reloadable
```

เมื่อโปรแกรมทำงาน `php start.php reload` จะส่งสัญญาณ reload (SIGUSR1) ไปยังกระบวนการย่อยทั้งหมด

กระบวนการย่อยที่ได้รับสัญญาณ reload จะออกโดยอัตโนมัติและกระบวนการหลักจะสร้างกระบวนการใหม่ขึ้นมา โดยทั่วไปใช้สำหรับการอับเดตโค้ดทางธุรกิจ

เมื่อคุณตั้งค่า $reloadable ของกระบวนการเป็นเท็จ หลังรับสัญญาณ reload จะเรียกใช้เฉพาะ [onWorkerReload](on-worker-reload.md) และจะไม่รีสตาร์ทกระบวนการปัจจุบัน

ตัวอย่างเช่น ในโมเดล Gateway/Worker การตั้งค่า reloadable ของกระบวนการ gateway เป็นเท็จ จะทำให้สามารถอับเดตโค้ดทางธุรกิจโดยไม่ต้องตัดสายการเชื่อมต่อของไคลเอ็นต์

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// ตั้งค่ารับสัญญาณ reload หลังจากนั้นให้รีสตาร์ทหรือไม่
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// รัน worker
Worker::runAll();
```
