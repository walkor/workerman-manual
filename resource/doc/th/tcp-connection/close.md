# ปิด
## คำอธิบาย:
```php
void Connection::close(mixed $data = '')
```

ปิดการเชื่อมต่ออย่างปลอดภัย

การเรียกใช้ close จะรอให้ข้อมูลในบัฟเฟอร์ส่งเสร็จก่อนที่จะปิดการเชื่อมต่อและเรียกใช้ ```onClose``` ในการใช้งาน

## พารามิเตอร์

 ``` $data ```

ตัวเลือกที่สามารถใส่ได้ คือ ข้อมูลที่ต้องการส่ง (หากมีการระบุโปรโตคอลจะทำการเรียกใช้เมธอด encode ของโปรโตคอลเพื่อแพ็คข้อมูล```$data```)，หลังจากส่งข้อมูลเสร็จจะทำการปิดการเชื่อมต่อ แล้วตามมาด้วยการเรียกใช้ onCLose ในการใช้งาน

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// รัน worker
Worker::runAll();
```
