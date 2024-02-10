# วิธีการดำเนินงานแบบไม่สะดุด

**คำถาม:**

วิธีการประมวลผลงานที่มีระบบเนื่องจากงานหนักๆ โดยใช้กระบวนการทำงานแบบไม่สะดุด ตัวอย่างเช่นการส่งอีเมลล์ให้กับ 1000 ผู้ใช้ กระบวนการนี้ช้ามาก อาจจะถูกบล็อกไว้เป็นเวลาหลายวินาที ในขณะนี้เนื่องจากกระบวนการหลักถูกบล็อก จะส่งผลต่อคำขอในภายหลัง วิธีที่จะให้งานหนักๆ เช่นนี้ไปยังกระบวนการในอีกกระบวนการอื่นๆ แบบไม่สะดุดได้อย่างไร

**ตอบ:**

คุณสามารถสร้างกระบวนการงานไว้ล่วงหน้าในเครื่องหรือเซิร์ฟเวอร์อื่น ๆ หรือกลุ่มเซิร์ฟเวอร์ โดยจำนวนของกระบวนการงานสามารถเปิดให้มากขึ้นได้เช่น การเปิดให้มากขึ้นเป็น10 เท่าของ CPU และจากนั้นอีกคำขอจะถูกส่งไปให้กระบวนการงานดำเนินการแบบการส่งข้อมูลแบบไม่สะดุด และรับผลลัพธ์จากกระบวนการงานแบบการส่งผลลัพธ์ที่ผ่านๆ มา

กระบวนการงานบริการดิบ
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// งานคนงานภาระ | ใช้โพรโตคอล Text
$task_worker = new Worker('Text://0.0.0.0:12345');
// จำนวนของการทำงานงานสามารถเปิดให้มากขึ้นตามความต้องการ
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // สมมติว่าข้อมูลที่ส่งมาเป็นJSON
     $task_data = json_decode($task_data, true);
     // ดำเนินการคำแนนตามข้อมูลงาน.... รับผลลัพธ์นี้และละเว้นส่วนนี้....
     $task_result = ......
     // ส่งผลลัพธ์
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

การเรียกใช้งานใน workerman

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// บริการผู้ใช้작เวบ
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // เชื่อมต่อกับบริการงานคนงานภาระบนเครื่องไกล ที่ IP ของบริการงานคนงานภาระของเครื่องไกล ถ้าเป็นบนเครื่องของตนเองคือ 127.0.0.1 ถ้าเป็นกลุ่มอื่นๆ จะเป็น IP ของ LVS
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // งานและข้อมูลพาราเมตเตอร์
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // ส่งข้อมูล
    $task_connection->send(json_encode($task_data));
    // รับผลลัพธ์แบบไม่สะดุด
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // ผลลัพธ์
         var_dump($task_result);
         // หลังจากได้รับผลลัพธ์ อย่าลืมปิดการเชื่อมต่อแบบไม่สะดุด
         $task_connection->close();
         // แจ้งเตือนผู้ใช้ websocket ที่เกี่ยวข้องว่าการทำงานเสร็จสมบูรณ์
         $ws_connection->send('task complete');
    };
    // ดำเนินการเชื่อมต่อแบบไม่สะดุด
    $task_connection->connect();
};

Worker::runAll();
```

ดังนั้น งานที่มีเนื่องจากระบวนการในเครื่องหรือเซิร์ฟเวอร์อื่น ๆ จะดำเนินการ ภาระหนัก หลังจากงานเสร็จจะได้รับผลลัพธ์แบบไม่สะดุด กระบวนการในภายหลังจะไม่ถูกบำบัด
