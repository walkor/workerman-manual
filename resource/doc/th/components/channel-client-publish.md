```php
# เผยแพร่
**``` (การต้องการ Workerman เวอร์ชัน >=3.3.0) ```**

```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
เผยแพร่เหตุการณ์ที่กำหนด ผู้ติดตามเหตุการณ์ทั้งหมดจะได้รับเหตุการณ์นี้และเรียกใช้การเรจิสเตอร์ที่ลงทะเบียนด้วย ```on($event_name, $callback)``` 

### พารามิเตอร์
 ``` $event_name ```

ชื่อเหตุการณ์ที่เผยแพร่ได้, สามารถเป็นสตริงใด ๆ หากไม่มีผู้ติดตามเหตุการณ์เหตุการณ์จะถูกละเว้น

 ``` $event_data ```

ข้อมูลที่เกี่ยวข้องกับเหตุการณ์, สามารถเป็นตัวเลข, สตริง หรืออาร์เรย์

### ค่าที่ส่งกลับ
หากเป็น void

### ตัวอย่าง
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
