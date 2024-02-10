# runAll
```php
function void Worker::runAll(void)
```
รันโปรแกรมทั้งหมดของ Worker instance ทุกตัว

**โน้ต:** 
หลังจากการทำ Worker::runAll() จะทำให้โปรแกรมติดอยู่อย่างต่อเนื่อง ก็คือโค้ดที่อยู่หลัง Worker::runAll() จะไม่ทำงาน การสร้าง Worker instance ทั้งหมดควรจะทำหลังจาก Worker::runAll()

### พารามิเตอร์
ไม่มีพารามิเตอร์

### ค่าที่คืน
ไม่มีค่าที่คืน

## ตัวอย่าง การรัน Worker instance หลายตัว

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// รัน Worker instance ทั้งหมด
Worker::runAll();
```


**โน้ต:** 
workerman เวอร์ชั่นสำหรับ Windows ไม่สนับสนุนการสร้าง Worker instance หลายตัวในไฟล์เดียวกัน
ตัวอย่างด้านบนไม่สามารถทำงานบน workerman เวอร์ชั่นสำหรับ Windows ได้

workerman เวอร์ชั่นสำหรับ Windows ต้องการให้การสร้าง Worker instance หลายตัวทำในไฟล์ที่แตกต่างกัน ดังตัวอย่างต่อไปนี้

start_http.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// รัน Worker instance ทั้งหมด(ในที่นี้มีเพียง instance เดียว)
Worker::runAll();
```


start_websocket.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// รัน Worker instance ทั้งหมด(ในที่นี้มีเพียง instance เดียว)
Worker::runAll();
```
