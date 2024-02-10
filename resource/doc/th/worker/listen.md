```php
public function listen(): void
```
ใช้สำหรับการเริ่มต้นการตรวจสอบหลังจากที่ทำการสร้าง Worker ไว้

เมธอดนี้ใช้สำหรับการสร้าง Worker ใหม่ๆ โดยหลังจากที่ Worker ถูกเริ่มต้นการทำงานแล้ว ซึ่งสามารถทำให้ Worker หนึ่ง ๆ สามารถเชื่อมต่อหลาย Port และรองรับโปรโตคอลต่าง ๆ กันได้ ควรทราบว่าการใช้เมธอดนี้จะทำให้ Worker เกิดการเพิ่มการตรวจสอบในกระบวนการปัจจุบันเท่านั้น และไม่สามารถสร้างกระบวนการใหม่ๆ หรือเรียกใช้เมธอด onWorkerStart ได้

ตัวอย่างเช่น หากมีการเริ่ม Worker แบบ http แล้วตามมาด้วยการสร้าง Worker แบบ websocket ใหม่ แทนที่ Worker จะสามารถทำการเข้าถึงผ่านโปรโตคอล http และ websocket เพราะ Worker แบบ websocket และ http จะอยู่ในกระบวนการเดียวกัน จึงสามารถเข้าถึงตัวแปรหน่วยความจำร่วมและการเชื่อมต่อ socket ได้ และสามารถรับคำขอ http แล้วตามด้วยการทำงานกับผู้ใช้ websocket ได้ในเวลาเดียวกัน

**โปรดทราบ:**

หาก PHP เวอร์ชัน <=7.0 จะไม่รองรับการสร้าง Worker ที่ใช้ Port เดิมในกระบวนการย่อยหลาย ๆ ตัว ตัวอย่างเช่น กระบวนการ A ที่สร้าง Worker ที่ฟังก์ชันที่ใช้ Port 2016 แล้ว กระบวนการ B จะไม่สามารถสร้าง Worker ที่ฟังก์ชันที่ใช้ Port 2016 เดียวกันได้ มิฉะนั้นจะเกิดข้อผิดพลาด ```Address already in use``` ตัวอย่างต่อไปนี้เป็นตัวอย่างที่ ```ไม่``` สามารถทำงานได้

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 กระบวนการ
$worker->count = 4;
// หลังจากที่ทุกระบวนการเริ่มต้น ลงไปสร้างการตรวจสอบ Worker ในกระบวนการปัจจุบัน
$worker->onWorkerStart = function($worker)
{
    /**
     * เมื่อมีการเริ่มต้นกระบวนการทั้ง 4 ก็จะสร้าง Worker ใน Port 2016
     * เมื่อสายลงไปที่ worker->listen() ก็จะเกิดความผิดพลาด Address already in use
     * หาก worker->count = 1 จะไม่เกิดข้อผิดพลาด
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // ทำการตรวจสอบ ที่นี่จะเกิดข้อผิดพลาด Address already in use
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// เริ่มทำงาน worker
Worker::runAll();
```

หาก PHP เวอร์ชัน>=7.0 คุณสามารถกำหนด Worker->reusePort=true ในการสร้าง Worker ที่ใช้ Port หลาย ๆ ตัว ตามตัวอย่างต่อไปนี้:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 กระบวนการ
$worker->count = 4;
// หลังจากที่ทุกระบวนการเริ่มต้น ลงไปสร้างการตรวจสอบ Worker ในกระบวนการปัจจุบัน
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // ตั้งการซ้ำ Port เพื่อให้สามารถสร้างการตรวจสอบเพื่อที่จะให้สามารถใช้ Port เดียวกันได้ (ต้องใช้ PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // ทำการตรวจสอบ โดยปกติจะไม่เกิดข้อผิดพลาด
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// เริ่มทำงาน worker
Worker::runAll();
```

### ตัวอย่างการส่งข้อมูลทันทีจากฝั่ง PHP Backend ไปยัง Client

**กำหนด:**

1. สร้าง Worker แบบ websocket เพื่อรักษาการเชื่อมต่อกับ Client ในระยะยาว
2. ภายใน Worker แบบ websocket สร้าง Worker แบบ text
3. Worker แบบ websocket และ text จะเป็นกระบวนการเดียวกัน สามารถแชร์การเชื่อมต่อกับ Client ได้
4. ระบบ PHP Backend ที่เป็นระบบอิสระสามารถสื่อสารผ่านโปรโตคอล text กับ Worker แบบ text
5. Worker แบบ text ดำเนินการเชื่อมต่อกับการแชร์ข้อมูลของ Worker แบบ websocket เพื่อการส่งข้อมูล

**รหัสและขั้นตอนการทำงาน**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// ตั้งค่า Worker แบบ websocket เพื่อฟังก์ชันที่ 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * ทำการตั้งค่าจำนวนกระบวนการเป็น 1
 */
$worker->count = 1;
// เมื่อกระบวนการ Worker เริ่ม จะสร้าง Worker แบบ text เพื่อเปิด Port แชร์
$worker->onWorkerStart = function($worker)
{
    // เริ่ม Port แชร์ ทำให้สามารถสื่อสารในระบบภายในเพื่อการส่งข้อมูลได้ รูปแบบข้อความ Text format รวมถึงตัวคั่นบรรทัด
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // รูปแบบข้อมูลเป็นอาร์เรย์ พร้อมข้อมูล uid แสดงถึงการส่งข้อมูลไปยังหน้าจอที่มี uid นั้น
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // ส่งข้อมูลไปยังหน้าจอตาม uid ที่กำหนด
        $ret = sendMessageByUid($uid, $buffer);
        // คืนค่าผลการส่ง
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## ทำการตรวจสอบ ##
    $inner_text_worker->listen();
};
// เพิ่มคุณลักษณะใหม่ ที่เก็บ uid ไปยังการเชื่อมต่อ
$worker->uidConnections = array();
// เมื่อ Client ส่งข้อความมา
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // ตรวจสอบว่า Client ถูกต้องหรือไม่ หรือใช่ว่าได้ตั้งค่า uid
    if(!isset($connection->uid))
    {
       // หากยังไม่ได้ตั้งค่า จะถือว่า package แรกคือ uid (ที่นี่ผมตั้งให้เป็นตัวอย่าง โดยที่ไม่ตรวจสอบอะไรเลย)
       $connection->uid = $data;
       /* บันทึก uid เชื่อมต่อไปยังการเชื่อมต่อ นั่นคือจะสามารถค้นหาการเชื่อมต่อตาม uid
        * เพื่อทำการส่่งข้อมูลตาม uid ได้
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// เมื่อ Client ยกเลิกการเชื่อมต่อ
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // ยกเลิกการเชื่อมต่อลบการผูกเชื่อมต่อ
        unset($worker->uidConnections[$connection->uid]);
    }
};

// ทำการส่งข้อมูลไปยังผู้ใช้ที่ได้ตรวจสอบ
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// ส่งข้อมูลไปยัง uid ที่กำหนด
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// เริ่มทำงาน worker ทั้งหมด
Worker::runAll();
```

เปิด Backend Service
 ```php push.php start -d```

โค้ด JS ที่ใช้รับการส่งข้อมูล
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

โค้ดการส่งข้อความจาก Backend
```php
// สร้างการเชื่อมต่อไปยังหน้าเชื่อมต่อภายในเพื่อทำการส่่งข้อมูล
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// ข้อมูลที่ต้องการส่งรวมถึง uid ที่แสดงถึงการส่งข้อมูลไป uid นั้น
$data = array('uid'=>'uid1', 'percent'=>'88%');
// ทำการส่งข้อมูล โปรดทราบว่า Port 5678 คือ Port ข้อมูล Text ข้อมูล Text ต้องเพิ่มเป็นการเว้นบรรทัด
fwrite($client, json_encode($data)."\n");
// อ่านผลการส่่งข้อมูล
echo fread($client, 8192);
```
