# การถ่ายโอน
## คำอธิบาย:
```php
string Worker::$transport
```

ตั้งค่าโปรโตคอลของ Worker ในปัจจุบันที่ใช้งาน เพียงเครื่องหมาย 3 แบบ (tcp、udp、ssl) เท่านั้น หากไม่ได้กำหนดจะถูกตั้งค่าเป็น tcp โดยค่าเริ่มต้น

``` หมายเหตุ: ต้องใช้ Workerman เวอร์ชั่น>=3.3.7 สำหรับ ssl ```

## ตัวอย่างที่ 1
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// ใช้โปรโตคอล udp
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// รัน worker
Worker::runAll();
```

## ตัวอย่างที่ 2
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ควรใช้ใบรับรองจากที่มีอยู่จริง
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // หรือ crt ไฟล์ก็ได้
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// กำหนดโปรโตคอลเป็น websocket ที่นี่
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// ตั้งค่าการถ่ายโอนเป็น ssl เพื่อเปิดใช้งาน ssl, websocket+ssl คือ wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
