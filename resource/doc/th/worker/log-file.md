# logFile
## คำอธิบาย:
```php
static string Worker::$logFile
```

ใช้เพื่อระบุตำแหน่งของไฟล์บันทึก log ของ workerman

ไฟล์นี้บันทึก log เกี่ยวกับ workerman เอง เช่น การเริ่มต้น หรือ การหยุดทำงาน

หากไม่ได้ระบุไว้ ชื่อไฟล์จะเป็น workerman.log และตำแหน่งของไฟล์จะอยู่ในโฟลเดอร์ที่อยู่ในระดับบนของ Workerman

**โปรดทราบ:** ไฟล์ log นี้จะบันทึกเฉพาะเรื่องการเริ่มหรือหยุดทำงานของ workerman เท่านั้น ไม่รวมถึง log ทางธุรกิจใดๆ

สำหรับ log ทางธุรกิจ สามารถใช้ฟังก์ชัน [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) หรือ [error_log](https://php.net/manual/zh/function.error-log.php) เพื่อสร้าง log ได้ตามที่ต้องการ

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// รัน worker
Worker::runAll();
```
