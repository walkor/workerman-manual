# user

## คำอธิบาย:
```php
string Worker::$user
```

กำหนดว่า Worker ในปัจจุบันจะทำงานด้วยผู้ใช้ใด โอนสิทธิ์ผู้ใช้เฉพาะเจาะจงสำหรับผู้ใช้ root เท่านั้น ถ้าไม่ได้กำหนดจะทำงานด้วยผู้ใช้ปัจจุบัน มีข้อแนะนำในการใช้ว่าควรกำหนด `$user` ให้กับผู้ใช้ที่มีสิทธิ์ต่ำ เช่น www-data, apache, nobody ฯลฯ

หมายเหตุ: คุณสมบัตินี้จะต้องกำหนดก่อนที่จะทำการรัน ```Worker::runAll();``` และระบบปฏิบัติการ Windows ไม่รองรับคุณสมบัตินี้

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// กำหนดผู้ใช้ให้กับ instance
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// รัน worker
Worker::runAll();
```
