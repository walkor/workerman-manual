# เชื่อมต่อ
**``` (ต้องการเวอร์ชั่นของ Workerman >= 3.3.0) ```**
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
เชื่อมต่อ Channel/Server

### พารามิเตอร์
 ``` listen_ip ```

ที่อยู่ IP ที่ Channel/Server ฟัง ถ้าไม่ระบุจะเป็น ```127.0.0.1```

 ``` listen_port ```

พอร์ตที่ Channel/Server ฟัง ถ้าไม่ระบุจะเป็น 2206

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
};

Worker::runAll();
```
