# ตัวอย่างที่ 1
**``` (ต้องการ Workerman เวอร์ชัน >= 3.3.0) ```**

ระบบการส่งข้อมูลแบบกระจายพูดเพิ่มชั้นเบิร์ดอยู่บน Worker ที่จ่ายซิสแบบหลายกระบวนการ (คลัสเตอร์การกระจาย) การกระจายแบบมวลภาพ เปิดให้ทุกโหนดต่อกัน
`start_channel.php`
หนึ่งระบบสามารถรันเพียงหนึ่งอันของ start_channel เท่านั้น ถูกคาดว่าจะใช้งานที่ 192.168.1.1
```php 
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// เริ่มบริการช่อง
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```
`start_ws.php`
ทั้งระบบสามารถใช้งาน start_ws ที่หลายๆอัน มีจุดสถานที่พักที่ 192.168.1.2 และ 192.168.1.3 บนเครื่องคอมพิวเตอร์สองตัว
```php 
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// บริการ websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // ผู้บริการ Channel เชื่อมต่อกับการบริการของ Channel
    Channel\Client::connect('192.168.1.1', 2206);
    // ด้วยการทำงานของด้วยตนเองที่มีเหตุการณ์ชื่อผู้บริการ id
    $event_name = $worker->id;
    // สมัครการรับเหรียญของผู้บริการด้วยตนเองและลงทะเบียนฟังก์ชันเหตุการณ์
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "การเชื่อมต่อไม่มีอยู่\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // สมัครการรับเหรียญส่องคล้อง once event
    $event_name = 'กระจาย';
    // เมื่อได้รับเหตุการณ์กระจายจะส่งข้อมูลกระจายไปยังในกระบวนการที่ลูกค้า   ทั้งสิ้น
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} connected\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```
`start_http.php`
ทั้งระบบสามารถใช้งาน start_ws ที่หลายๆอัน ที่ตำแหน่งที่พักของแต่ละตัวที่ 192.168.1.4 และ 192.168.1.5
```php 
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ใช้ในการจัดการคำขอ http  รวมไปถึงการแจ้งทุกโหนด​
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // ทำให้ code ทำงานร่วมกับ workerman4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // คือการแจ้งข้อมูลไปยังโครงสร้าง การประมาณงานคน
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // คือการส่งข้อมูล ทั่งหมดจุดยอดภาพ
    else
    {
        $event_name = 'กระจาย';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```
## การทดสอบ 
1. เริ่มแต่ละบริการบนเซิร์ฟเวอร์
2. การเชื่อมต่อคลายแต่ละลูกค้า
ให้เปิดเบราว์เซอร์ Chrome และกด F12 เพื่อเปิดหน้าต่างห้องควบคุม ที่แหล่งข้อมูล บริการ Console พิมพ์ข้อความต่อไปนี้เข้าไป (หรือใส่โค้ดต่อไปนี้เข้าไปในหน้าเว็บ html แล้วรันด้วย JavaScript)

```javascript 
// สามารถเชื่อมต่อ ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("ได้รับข้อความจากเซิร์ฟเวอร์：" + e.data);
}; 
```

3. ผ่านการเรียกร้อง HTTP
เข้าถึง url ```http://192.168.1.4:4237/?content={$content}``` หรือ ```http://192.168.1.5:4237/?content={$content}``` ที่จะนำ```$content```ไปยังการเชื่อมต่อลูกค้าทุกโหนด 

เข้าถึง url ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` หรือ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` ที่จะนำ```$content```ไปยังการเชื่อมต่อลูกค้าแต่ละโนดที่แสดง

โปรดทราบ: ขณะทดสอบ กรุณาแทนที่ ```{$worker_id}``` ```{$connection_id}``` และ ```{$content}``` ด้วยค่าจริง
