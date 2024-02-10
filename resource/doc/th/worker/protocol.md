# โปรโตคอล

ต้องการ ```(workerman >= 3.2.7)```

## คำอธิบาย:
```php
string Worker::$protocol
```

การตั้งค่าคลาสโปรโตคอลของ Worker ปัจจุบัน

หมายเหตุ: คลาสการจัดการโปรโตคอลสามารถระบุโดยตรงในพารามิเตอร์การรับฟังก์ชั่นของ Worker ในขณะที่กำลังเริ่มต้น เช่น
```php
$worker = new Worker('http://0.0.0.0:8686');
```

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// รัน worker ทั้งหมด
Worker::runAll();
```

โค้ดข้างต้นเทียบเท่ากับโค้ดด้านล่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * ก่อนอื่นจะตรวจสอบว่าผู้ใช้มีคลาสโปรโตคอล\Protocols\Http ที่กำหนดเองหรือไม่
 * หากไม่มี จะใช้คลาสโปรโตคอลภายในของ workerman คือ Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// รัน worker ทั้งหมด
Worker::runAll();
```
