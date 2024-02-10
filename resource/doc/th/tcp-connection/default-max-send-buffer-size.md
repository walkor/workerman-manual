# defaultMaxSendBufferSize
## คำอธิบาย:

```php
static int Connection::$defaultMaxSendBufferSize
```

สมบัตินี้เป็นสมบัติแบบ global static ที่ใช้สำหรับตั้งค่าขนาดของบัฟเฟอร์การส่งข้อมูลที่เป็นค่าเริ่มต้นสำหรับการเชื่อมต่อทั้งหมด หากไม่ได้ตั้งค่า ค่าเริ่มต้นคือ ```1MB```  ```Connection::$defaultMaxSendBufferSize``` สามารถตั้งค่าได้แบบ dynamic โดยที่การตั้งค่าจะมีผลเฉพาะกับการเชื่อมต่อใหม่ที่จะเกิดขึ้นในภายหลัง


สมบัตินี้จะมีผลต่อ [onBufferFull](../worker/on-buffer-full.md) callback.


## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ตั้งค่าขนาดของบัฟเฟอร์การส่งข้อมูลที่เป็นค่าเริ่มต้นสำหรับการเชื่อมต่อทั้งหมด
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // ตั้งค่าขนาดของบัฟเฟอร์การส่งข้อมูลที่เป็นค่าเริ่มต้นสำหรับการเชื่อมต่อปัจจุบัน ซึ่งจะทับท่าค่าเริ่มต้น
    $connection->maxSendBufferSize = 4*1024*1024;
};
// รัน worker
Worker::runAll();
```
