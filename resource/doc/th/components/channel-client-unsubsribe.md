# ยกเลิกการสมัครสมาชิก

**``` (ต้องการ Workerman เวอร์ชัน >= 3.3.0) ```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```

ยกเลิกการสมัครสมาชิกในเหตุการณ์ที่เกิดขึ้น ก่อนหน้านี้จะไม่สามารถเรียกใช้การเรียกคืน ```on($event_name, $callback)``` ที่ลงทะเบียนไว้ ```$callback```

### พารามิเตอร์
 ``` $event_name ```

ชื่อเหตุการณ์

### ค่าที่ส่งกลับ
void

### ตัวอย่าง
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
