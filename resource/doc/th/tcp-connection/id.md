# ไอดี

## คำอธิบาย:
```php
int Connection::$id
```

ไอดีของการเชื่อมต่อ นี่คือจำนวนเติบต่อที่เพิ่มขึ้นเอง

หมายเหตุ: workerman เป็นกระบวนการหลายอัน แต่ละกระบวนการจะบำรุงรักษาไอดีของการเชื่อมต่อที่เพิ่มขึ้นเอง ดังนั้นไอดีการเชื่อมต่อจะซ้ำกันระหว่างกระบวนการหลายอัน หากต้องการไอดีการเชื่อมต่อที่ไม่ซ้ำกัน สามารถให้ค่าใหม่ให้กับ connection->id ตามต้องการ เช่น ให้เพิ่มคำนำหน้า worker->id

## ดูเพิ่มเติม
[คุณสมบัติ connections ของ Worker](../worker/connections.md)


## ตัวอย่าง


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// รัน worker
Worker::runAll();
```
