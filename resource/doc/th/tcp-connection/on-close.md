# การปิดการเชื่อมต่อ (onClose)

## คำอธิบาย:
```php
callback Connection::$onClose
```

การเรียกคืน (callback) นี้มีผลเหมือนกับการเรียกคืน [Worker::$onClose](../worker/on-close.md) แต่มีความแตกต่างตรงที่มีผลเฉพาะกับการเชื่อมต่อปัจจุบันเท่านั้น ก็คือสามารถกำหนด callback onClose สำหรับการเชื่อมต่อใดใดได้

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// เมื่อเหตุการณ์การเชื่อมต่อเกิดขึ้น
$worker->onConnect = function(TcpConnection $connection)
{
    // ตั้งค่า callback onClose ของการเชื่อมต่อ
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connection closed\n";
    };
};
// ทำงาน worker
Worker::runAll();
```

โค้ดด้านบนเหมือนกับผลลัพธ์ด้านล่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// ตั้งค่า callback การปิดการเชื่อมต่อของทุกการเชื่อมต่อ
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// ทำงาน worker
Worker::runAll();
```
