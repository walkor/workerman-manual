# โปรโตคอล

## คำอธิบาย:
```php
string Connection::$protocol
```

กำหนดคลาสโปรโตคอลของการเชื่อมต่อปัจจุบัน


## ตัวอย่าง


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // การส่งข้อมูลจะเรียกใช้ $connection->protocol::encode() โดยอัตโนมัติ แบ่งข้อมูลก่อนส่ง
    $connection->send("hello");
};
// เรียกใช้ worker
Worker::runAll();
```
