# stdoutFile
## คำอธิบาย:
```php
static string Worker::$stdoutFile
```

คุณสมบัตินี้เป็นคุณสมบัติสถิตทั่วไป หากทำการรันด้วยวิธี Daemon mode (เริ่มต้นด้วย ```-d```) การเอาเอาท์ทั้งหมด (เช่น echo var_dump เป็นต้น) จะถูก redirect ไปยังไฟล์ที่ได้รับการกำหนดที่ stdoutFile

หากไม่ได้กำหนด และรันด้วยวิธี Daemon mode การเอาเอาทั้งหมดจะถูก redirect ไปที่ `/dev/null` (ก็คือการละทิ้งทุกการเอาเอาท์เรียบร้อย)

> โปรดทราบ: `/dev/null` เป็นไฟล์พิเศษในระบบลีนุกซ์ จริง ๆ แล้วมันคือหลุมดำ การเขียนข้อมูลเข้าไปในไฟล์นี้จะถูกละทิ้งทั้งหมด หากไม่ต้องการที่จะละทิ้งเอาท์พวกนี้ คุณสามารถกำหนด `Worker::$stdoutFile` เป็นเส้นทางของไฟล์ที่ถูกต้อง

> โปรดทราบ: คุณสมบัตินี้ต้องถูกกำหนดก่อนที่จะทำการรัน ```Worker::runAll();``` ระบบปฏิบัติการวินโดวส์ไม่สนับสนุนคุณสมบัตินี้

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// การเอาเอาทั้งหมดถูกบันทึกไว้ที่ไฟล์ /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "เริ่มต้น Worker\n";
};
// ทำไห้ worker ทำงาน
Worker::runAll();
```
