# onMessage
## คำอธิบาย:
```php
callback Worker::$onMessage
```

เมื่อมีการส่งข้อมูลผ่านการเชื่อมต่อจากไคลเอ็นต์ (เมื่อ Workerman ได้รับข้อมูล) จะเรียกใช้ callback function

## พารามิเตอร์ของ callback function

 ``` $connection ```

Object ของการเชื่อมต่อ หรือ [TcpConnection อินสแตนซ์](../tcp-connection.md) ที่ใช้ในการดำเนินการกับการเชื่อมต่อของไคลเอ็นต์ เช่น [ส่งข้อมูล](../tcp-connection/send.md) หรือ [ปิดการเชื่อมต่อ](../tcp-connection/close.md) เป็นต้น

 ``` $data ```

ข้อมูลที่ไคลเอ็นต์ส่งมาทางการเชื่อมต่อ หาก Worker กำหนดโปรโตคอลไว้แล้ว ข้อมูลจะเป็นข้อมูลที่ถอดรหัสตามโปรโตคอลโดยใช้ `decode()` ประเภทข้อมูลขึ้นอยู่กับการถอดรหัสโดยโปรโตคอล เช่น `websocket` `text` `frame` จะเป็นข้อความ ส่วนข้อมูลจากโปรโตคอล HTTP จะเป็นอ็อบเจกต์ [`Workerman\Protocols\Http\Request`](../http/request.md)

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// รัน worker
Worker::runAll();
```

เพิ่มเติม: นอกจากใช้ฟังก์ชั่นแบบไม่ระบุชื่อเป็น callback ยังสามารถ[ดูตัวอย่างเพิ่มเติมได้ที่นี่](../faq/callback_methods.md)
