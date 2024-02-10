# getRemoteIp
## รายละเอียด:
```php
string Connection::getRemoteIp()
```

ดึง IP ของ client ที่เชื่อมต่อนี้

## พารามิเตอร์

ไม่มีพารามิเตอร์

## ตัวอย่าง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "การเชื่อมต่อใหม่จาก IP " . $connection->getRemoteIp() . "\n";
};
// เรียกใช้ worker
Worker::runAll();
```
