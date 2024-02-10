# worker
## คำอธิบาย:
```php
Worker Connection::$worker
```

คุณสมบัตินี้เป็นคุณสมบัติที่อ่านได้อย่างเพียงอย่างเดียว นั่นคือ worker ตัวอย่างปัจจุบันที่เป็นเจ้าของวัสดุนี้


## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// เมื่อมีข้อมูลถูกส่งมาจาก client คนหนึ่ง ให้ส่งต่อไปยัง client ทั้งหมดที่ worker ปัจจุบันกำลังดูแล
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// รัน worker ทั้งหมด
Worker::runAll();
```
