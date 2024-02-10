# ท่อ
## คำอธิบาย:
```php
void Connection::pipe(TcpConnection $target_connection)
```



## พารามิเตอร์
นำการสื่อสารของการเชื่อมต่อปัจจุบันไปยังการเชื่อมต่อเป้าหมาย มีการควบคุมการไหลตามภายใน วิธีนี้เป็นประโยชน์มากสำหรับการทำ TCP พร็อกซี่



## ตัวอย่าง TCP พร็อกซี่

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// เมื่อเชื่อมต่อ tcp หลังจากนั้น
$worker->onConnect = function(TcpConnection $connection)
{
    // เชื่อมต่อแบบไม่สม่ำเสมอที่พอร์ต 80 ที่อยู่ตามหลังคา
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // ตั้งค่าส่งการติดต่อของลูกค้าปัจจุบันไปที่การเชื่อมต่อที่พอร์ต 80
    $connection->pipe($connection_to_80);
    // ตั้งค่าการติดต่อของพอร์ต 80 ที่มีการส่งกลับของข้อมูลไปยังการเชื่อมต่อของลูกค้า
    $connection_to_80->pipe($connection);
    // เริ่มการเชื่อมต่อแบบไม่สม่ำเสมอ
    $connection_to_80->connect();
};

// ให้ Worker ทำงานทั้งหมด
Worker::runAll();
```
