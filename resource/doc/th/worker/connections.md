# การเชื่อมต่อ
## คำอธิบาย:
```php
array Worker::$connections
```

รูปแบบคือ
```php
array(id=>connection, id=>connection, ...)
```

คุณสมบัตินี้เก็บวัตถุการเชื่อมต่อของ**กระบวนการปัจจุบัน**ทั้งหมดของลูกค้า โดยที่ id คือ หมายเลข id ของการเชื่อมต่อ connection ดูรายละเอียดได้ที่[คุณสมบัติ id ของ TcpConnection](../tcp-connection/id.md)。

```$connections``` มีประโยชน์มากในการกระจายข้อมูลหรือในการรับวัตถุการเชื่อมต่อตาม id ที่กำหนดไว้

หากทราบว่าหมายเลขการเชื่อมต่อคือ ```$id``` คุณสามารถหาวัตถุการเชื่อมต่อที่สอดคล้องกันได้อย่างสะดวกผ่าน```$worker->connections[$id]``` และจากนั้นคุณสามารถดำเนินการกับการเชื่อมต่อของ socket ที่สอดคล้องกัน เช่น ส่งข้อมูลผ่าน ```$worker->connections[$id]->send('...')``` 

โปรดทราบ: หากการเชื่อมต่อถูกปิด (กระตุ้น onCLose) จะลบรายการ```connection``` สอดคล้องกันออกจากอาร์เรย์```$connections``` 

ประณณวิธี : นักพัฒนาไม่ควรดำเนินการแก้ไขคุณสมบัตินี้เพราะอาจทำให้เกิดสถานการณ์ที่ไม่สามารถคาดเดาได้

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// เมื่อกระบวนการปัจจุบันเริ่มต้นกำหนดตัวนับเวลาที่จะส่งข้อมูลไปที่ลูกค้าทุกคน
$worker->onWorkerStart = function($worker)
{
    // ตั้งเวลา ทุก 10 วินาที
    Timer::add(10, function()use($worker)
    {
        // วิ่งทั้งหมดของการเชื่อมต่อลูกค้าทุกตัว และส่งเวลาของเซิฟเวอร์ทันที
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// รัน worker
Worker::runAll();
```
