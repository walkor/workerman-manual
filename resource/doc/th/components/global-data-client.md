# ไคลเอ็นต์ GlobalData คอมโพเนนท์
**``` (ต้องการ Workerman เวอร์ชั่น >= 3.3.0) ```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

สร้างอ็อบเจ็กต์ไคลเอ็นต์ \GlobalData\Client ขึ้นมา โดยการกำหนดค่าแอตทริบิวต์บนอ็อบเจ็กต์ไคลเอ็นต์เพื่อแชร์ข้อมูลระหว่างโพรเซส

### พารามิเตอร์
GlobalData server ที่อยู่ของเซิร์ฟเวอร์ รูปแบบ ```<ip address>:<พอร์ต>``` เช่น ```127.0.0.1:2207``` 

หากเป็นคลัสเตอร์เซิร์ฟเวอร์ GlobalData ให้ส่งอาร์เรย์ของที่อยู่ เช่น ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```

## คำอธิบาย
รองรับการกำหนดค่า อ่าน ตรวจสอบว่ามีหรือไม่ และถอนการกำหนดค่า
รองรับการดำเนินการ cas แบบอะตอมิกพร้อมกัน

## ตัวอย่าง

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// เซิร์ฟเวอร์ GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// เมื่อโพรเซสเริ่มทำงาน
$worker->onWorkerStart = function()
{
    // กำหนดค่าไคลเอ็นต์ GlobalData ให้กลายเป็นอ็อบเจ็กต์กลุ่มทั้งหมด
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// ทุกครั้งที่เซิร์ฟเวอร์รับข้อความ
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // เปลี่ยนค่าของ $global->somedata ค่านี้ โพรเซสอื่นจะได้รับการแชร์ตัวแปร $global->somedata
    global $global;
    echo "now global->somedata=".var_export($global->somedata, true)."\n";
    echo "set \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### การใช้งานทั้งหมด (สามารถใช้ในสภาพแวดล้อม php-fpm ได้)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## ข้อควรระวัง:
คอมโพเนนท์ GlobalData ไม่สามารถแชร์ข้อมูลประเภททรัพยากร เช่น การเชื่อมต่อ mysql, การเชื่อมต่อ socket ไม่สามารถแชร์ได้

หากใช้คลาส GlobalData/Client ในระบบ Workerman โปรดสร้างอ็อบเจ็กต์ GlobalData/Client ในคอลแบ็กซ์ onXXX เช่น สร้างใน onWorkerStart

ไม่สามารถดำเนินการแชร์ตัวแปรแบบนี้
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
สามารถดำเนินการแชร์ตัวแปรแบบนี้
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
