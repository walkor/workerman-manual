# onMessage
## คำอธิบาย:

```php
callback Connection::$onMessage
```

เป็นกลไกเดียวกับการเรียกใช้ [Worker::$onMessage](../worker/on-message.md) โดยทำงานต่างกันแค่ว่ามีผลเฉพาะต่อการเชื่อมต่อปัจจุบันเท่านั้น ก็คือสามารถตั้งค่าการเรียกใช้ onMessage callback สำหรับการเชื่อมต่อที่เฉพาะเจาะจง

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// เมื่อเกิดเหตุการณ์การเชื่อมต่อของ client
$worker->onConnect = function(TcpConnection $connection)
{
    // ตั้งค่าการเรียกใช้ onMessage callback ของการเชื่อมต่อ
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('receive success');
    };
};
// ทำงาน worker
Worker::runAll();
```

โค้ดด้านบนเหมือนกับโค้ดด้านล่างนี้

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// ตั้งค่าการเรียกใช้ onMessage callback ของการเชื่อมต่อทั้งหมดโดยตรง
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// ทำงาน worker
Worker::runAll();
```
