# getRemotePort
## คำอธิบาย:
```php
int Connection::getRemotePort()
```

รับพอร์ตของไคลเอนต์ของการเชื่อมต่อ

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
    echo "การเชื่อมต่อใหม่จากที่อยู่ " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// เริ่มการทำงานของ worker
Worker::runAll();
```
