# วิธีการส่งข้อมูลแบบส่งข่าว (Broadcast)

## ตัวอย่าง (การส่งข่าวตามเวลา)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// ตัวอย่างนี้จำเป็นต้องมีจำนวนกระบวนการเป็น 1
$worker->count = 1;
// เมื่อกระบวนการเริ่มทำงานจะตั้งตัวจับเวลาส่งข้อมูลไปยังการเชื่อมต่อของผู้ใช้ทุกคนทุกๆ 10 วินาที
$worker->onWorkerStart = function($worker)
{
    // ตั้งเวลา ทุก 10 วินาที
    Timer::add(10, function()use($worker)
    {
        // วนซ้ำการเชื่อมต่อของผู้ใช้ทั้งหมดของกระบวนการปัจจุบัน และส่งเวลาปัจจุบันของเซิร์ฟเวอร์ไป
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// รัน worker
Worker::runAll();
```

## ตัวอย่าง (การพูดคุยแบบกลุ่ม)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// ตัวอย่างนี้จำเป็นต้องมีจำนวนกระบวนการเป็น 1
$worker->count = 1;
// เมื่อมีข้อความเข้ามาจากผู้ใช้ จะส่งข้อความไปยังผู้ใช้อื่นๆ
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// รัน worker
Worker::runAll();
```

## คำอธิบาย:
**กระบวนการเดียว:**
ตัวอย่างข้างต้นสามารถทำงานได้เพียง**กระบวนการเดียว** เท่านั้น (```$worker->count=1```) เนื่องจากเมื่อมีหลายกระบวนการ ผู้ใช้หลายคนอาจเชื่อมต่อไปยังกระบวนการที่ต่างกัน การเชื่อมต่อระหว่างกระบวนการจะเป็นอิสระจึงไม่สามารถสื่อสารโดยตรง กล่าวคือ กระบวนการ A ไม่สามารถดำเนินการส่งข้อมูลโดยตรงไปยังรายการเชื่อมต่อของกระบวนการ B (เพื่อทำได้นี้ ต้องมีการสื่อสารระหว่างกระบวนการ เช่น อาจใช้ Channel คอมโพเน้นต์ เช่น [ตัวอย่าง-การส่งไปยัม้ย](../components/channel-examples.md) หรือ[ตัวอย่าง-การส่งแบบกลุ่ม](../components/channel-examples2.md))

**แนะนำให้ใช้ GatewayWorker**
GatewayWoker ที่พัฒนาขึ้นจาก workerman ให้การปฏิบัติการส่งที่สะดวกมากขึ้น เช่น ส่งแบบกลุ่ม การส่งข่าว และอื่นๆ ส่งไปยังหลายกระบวนการ หรือการติดตั้งบนหลายเซิร์ฟเวอร์ หากต้องการส่งข้อมูลไปยังผู้ใช้ แนะนำให้ใช้ GatewayWorker framework

URL คู่มือ GatewayWorker https://doc2.workerman.net
