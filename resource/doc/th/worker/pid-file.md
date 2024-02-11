# ไฟล์ pidFile
## คำอธิบาย:
```php
static string Worker::$pidFile
```
ถ้าไม่มีความจำเป็น เราขอแนะนำไม่ตั้งค่าคุณสมบัตินี้

นี่คือคุณสมบัติทั่วไปแบบสถิต ที่ใช้สำหรับกำหนดเส้นทางของไฟล์ pid ของ Workerman โดยพื้นคุณก็คือ ในการติดตามการทำงาน ตัวอย่างเช่น การใส่ไฟล์ pid ของ Workerman ไว้ในโฟลเดอร์ที่คงที่ สามารถช่วยให้ซอฟแวร์ติดตามอ่านไฟล์ pid และติดตามสถานะของกระบวนการ Workerman ได้อย่างสะดวก

หากไม่ตั้งค่า Workerman จะสร้างไฟล์ pid โดยอัตโนมัติมาไว้ที่ตำแหน่งเดียวกับไดเรกทอรี Workerman ($sys_get_temp_dir() เป็นค่าเริ่มต้นในเวอร์ชั่นก่อน 3.2.3) และเพื่อหลีกเลี่ยงปัญหาของการเริ่มต้นหลายๆ อินสแตนซ์ของ Workerman ซึ่งจะทำให้เกิดข้อขัดแย้งในไฟล์ pid Workerman จะเพิ่มเส้นทางไปยัง Workerman ปัจจุบันด้วย

หมายเหตุ: คุณสมบัตินี้จะมีผลเมื่อถูกตั้งค่าก่อน ```Worker::runAll();``` ระบบปฏิบัติการของ Windows ไม่รองรับคุณสมบัตินี้

## ตัวอย่าง

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "เริ่ม Worker";
};
// เริ่ม worker
Worker::runAll();
```
