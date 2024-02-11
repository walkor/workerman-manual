# การถือครองวัตถุและทรัพยากร
ในการพัฒนาเว็บแบบดั้งเดิม วัตถุ เช่น ข้อมูล และทรัพยากรที่สร้างขึ้นโดย PHP จะถูกปล่อยให้เสร็จสิ้นการขอใช้หลังจากการร้องขอ ซึ่งทำให้เกิดความยากลำบากในการถือครองอย่างยั่งยืน แต่ใน Workerman สามารถทำได้อย่างง่ายดาย

ใน Workerman หากต้องการที่จะถือครองข้อมูลทรัพยากรบางอย่างไว้ในหน่วยความจำได้โดยการใส่ทรัพยากรไว้ในตัวแปร global หรือสมาชิกสถิตของคลาส

ตัวอย่างโค้ดด้านล่างนี้:

ใช้ตัวแปร global ```$connection_count``` เพื่อเก็บจำนวนการเชื่อมต่อของไคลเอนท์สำหรับกระบวนการปัจจุบัน

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ตัวแปร global เพื่อเก็บจำนวนการเชื่อมต่อของไคลเอนท์สำหรับกระบวนการปัจจุบัน
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // เมื่อมีการเชื่อมต่อของไคลเอนท์ใหม่ เพิ่มจำนวนการเชื่อมต่อไปที่ 1
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // เมื่อไคลเอนท์ปิดการเชื่อมต่อ ลดจำนวนการเชื่อมต่อลง 1
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};

```


## ขอบเขตของตัวแปร PHP ดูที่นี่:
https://php.net/manual/zh/language.variables.scope.php
