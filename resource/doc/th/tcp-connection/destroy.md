# ทำลาย
## คำอธิบาย:
```php
void Connection::destroy()
```

ปิดการเชื่อมต่อทันที

ความแตกต่างของการเรียกใช้ destroy คือ การเชื่อมต่อจะถูกปิดทันทีแม้ว่าข้อมูลในบัฟเฟอร์การส่งของการเชื่อมต่อยังไม่ได้ถูกส่งไปยังอีกฝั่ง และจะเรียกใช้ ```onClose``` callback ของการเชื่อมต่อทันที

## พารามิเตอร์

ไม่มีพารามิเตอร์

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// รัน worker
Worker::runAll();
```
