# maxSendBufferSize
## คำอธิบาย:
```php
int Connection::$maxSendBufferSize
```

ทุกครั้งที่เชื่อมต่อจะมีบัฟเฟอร์ส่งขึ้นแอปพลิเคชันเท่าไหร่ หากอุปกรณ์ของลูกค้าให้ความเร็วในการรับไม่ได้เท่ากับความเร็วในการส่งของเซิร์ฟเวอร์ ข้อมูลจะถูกเก็บไว้ในบัฟเฟอร์ของแอปพลิเคชันรอการส่ง

คุณสมบัตินี้ใช้ในการกำหนดขนาดของบัฟเฟอร์ส่งของแอปพลิเคชันสำหรับการเชื่อมต่อปัจจุบัน หากไม่ได้กำหนด ค่าเริ่มต้นคือ [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md) (1MB)

คุณสมบัตินี้มีผลต่อการเรียกใช้ [onBufferFull](../worker/on-buffer-full.md) callback


## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // ตั้งค่าขนาดบัฟเฟอร์ส่งของการเชื่อมต่อปัจจุบันเป็น 102400 ไบต์
    $connection->maxSendBufferSize = 102400;
};
// ให้ worker ทำงาน
Worker::runAll();
```
