# การเปลี่ยนให้เป็น Daemonize
## คำอธิบาย:
```php
static bool Worker::$daemonize
```

คุณสมบัตินี้เป็นคุณสมบัติแบบสถานะทั่วไป ซึ่งแสดงถึงว่าโปรแกรมรันอยู่ในโหมด Daemon (เซิร์ฟเวอร์หรือบริการพื้นหลัง) หรือไม่ หากคำสั่งเริ่มต้นใช้พารามิเตอร์ ```-d``` คุณสมบัตินี้จะถูกตั้งค่าเป็น true สามารถตั้งค่าด้วยโค้ดเองได้เช่นกัน

หมายเหตุ: คุณสมบัตินี้ต้องถูกตั้งค่าก่อนที่ ```Worker::runAll();``` จึงจะมีผล ระบบปฏิบัติการ windows ไม่รองรับคุณสมบัตินี้

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// รัน worker
Worker::runAll();
```
