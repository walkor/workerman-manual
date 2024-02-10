# วิธีการส่งข้อความอย่างเป็นอิสระ

1. คุณสามารถใช้ตัวจับเวลาเพื่อส่งข้อมูลเป็นระยะ ๆ ได้
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// เมื่อเริ่มต้นกระบวนการพร้อม จะส่งข้อมูลไปยังไคลเอนต์ตามเวลาที่กำหนด
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

2. เมื่อเกิดเหตุการณ์ใดก็ตามในโปรเจกต์อื่น ๆ ที่ต้องการแจ้ง workerman เพื่อส่งข้อมูล
ดูเพิ่มเติมที่ [คำถามที่พบบ่อย-การส่งข้อมูลจากโปรเจกต์อื่น](push-in-other-project.md)

