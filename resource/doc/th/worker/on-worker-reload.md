# onWorkerReload
ต้องการ ```(workerman >= 3.2.5)```
## คำอธิบาย:
```php
callback Worker::$onWorkerReload
```
คุณสมบัตินี้ไม่ได้ใช้บ่อย ๆ

ตั้งค่าการเรียกคืนหลังจาก Worker ได้รับสัญญาณ reload

คุณสามารถใช้งาน onWorkerReload callback เพื่อทำสิ่งมากมาย เช่น โหลดไฟล์ตั้งค่าธุรกิจใหม่โดยไม่ต้องรีสตาร์ตกระบวนการ

**โปรดทราบ**：

พฤติกรรมที่เป็นค่าเริ่มต้นสำหรับกระบวนการย่อยหลังจากได้รับสัญญาณ reload คือ การออกและรีสตาร์ตเพื่อง่ายต่อกระบวนการใหม่ที่จะโหลดโค้ดธุรกิจเพื่อการปรับปรุงโค้ด ดังนั้นหลังจากที่กระบวนการย่อยแสดงคำเตือนดำเนินการออกจาก workerman. หลังจากการเรียกคืนหลังจาก Worker ทำ  callback แล้ว จะทำการออกทันทีคือสิ่งปกติ

หากต้องการให้กระบวนการย่อยทำการ callback หลังจากได้รับสัญญาณ reload และไม่ต้องการออก สามารถตั้งค่า  reloadable property ของ Worker instance ที่เกี่ยวข้องเป็น false ได้

## พารามิเตอร์ของฟังก์ชัน callback
 ``` $worker ```

 สิ่งนี้คือว่าเครื่องเซิฟเวอร์

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// ตั้งค่า reloadable เป็นเท็จ หมายความว่ากระบวนการย่อยที่ได้รับสัญญาณ reload จะไม่ทำการรีสตาร์ต
$worker->reloadable = false;
// หลังจากที่มีการรีโหลแล้วบอกลูกค้าทุกคนว่ามีการรีโหล
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// รันเซิฟเวอร์
Worker::runAll();
```

เรียกใช้ฟังก์ชันอนามัยเป็น callback แทนการใช้ฟังก์ชันอื่น ๆ สามารถเข้าถึงได้ที่นี่บ้าง：[ที่นี่](../faq/callback_methods.md)
