# นับ

## คำอธิบาย:
```php
int Worker::$count
```

กำหนดจำนวนโปรเซสที่ Worker จะเริ่มต้นขึ้นมากี่ตัว ถ้าไม่ได้ตั้งค่าไว้ จะมีค่าเริ่มต้นเท่ากับ 1

如何设置进程数，请参考[**这里**](../faq/processes-count.md) 。
หมายเหตุ: คุณต้องตั้งค่านี้ก่อนที่ ```Worker::runAll();``` เพื่อให้มีผล

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// เริ่มต้นโปรเซส 8 ตัว และให้ฟังพอร์ต 8484 พร้อมให้บริการโดยใช้โปรโตคอล websocket
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// รัน worker ทั้งหมด
Worker::runAll();
```
