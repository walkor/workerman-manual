# stopAll
```php
void Worker::stopAll(void)
```

หยุดกระบวนการปัจจุบันและออกไป

> **หมายเหตุ**
> `Worker::stopAll()` ใช้ในการหยุดกระบวนการปัจจุบัน หลังจากนั้น main process จะทันทีสร้างกระบวนการใหม่ขึ้นมา หากคุณต้องการหยุดการทำงานของทั้งระบบ workerman โปรดเรียกใช้ `posix_kill(posix_getppid(), SIGINT)`

### พารามิเตอร์
ไม่มีพารามิเตอร์

### ค่าที่ส่งกลับ
ไม่มีค่าที่ส่งกลับ

## ตัวอย่าง max_request

ในตัวอย่างต่อไปนี้ หลังจากที่ sub process ได้ประมวลผลคำขอไป 1000 ครั้ง จะทำการ stopAll เพื่อออกจากกระบวนการ และสร้างกระบวนการใหม่ขึ้นมาเพื่อทำงานอีกครั้ง คล้ายกับคุณสมบัติ max_request ใน php-fpm ซึ่งถูกใช้เพื่อแก้ปัญหาหน่วยความจำที่รั่วไหลที่เกิดจากบั๊กในโค้ดธุรกิจของ php

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ให้แต่ละกระบวนการทำงานสูงสุดเป็น 1000 คำขอ
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // จำนวนคำขอที่ถูกประมวลผลแล้ว
    static $request_count = 0;

    $connection->send('hello http');
    // หากจำนวนคำขอมากกว่าหรือเท่ากับ 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * ออกจากกระบวนการปัจจุบัน main process จะทันทีสร้างกระบวนการใหม่ขึ้นมาเพื่อให้ปิดทองกระบวนการ
         * เพื่อทำการรีสตาร์ทกระบวนการ
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
