เว็บแมน - การส่งข้อมูลไปยังไคลเอนต์ที่ระบุไว้ใน Workerman
การใช้ worker เพื่อทำเซิร์ฟเวอร์ โดยไม่ใช้ GatewayWorker นั่นคือวิธีการที่ใดที่ซึ่งสามารถทำการส่งข้อมูลไปยังผู้ใช้ที่กำหนดได้?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// กำหนด worker container เพื่อฟังก์ชันการทำงานได้ในพอร์ต 1234
$worker = new Worker('websocket://workerman.net:1234');
// ==== จำนวนของกระบวนการทำงานจำเป็นต้องตั้งค่าเป็น 1 นี้ ======================
$worker->count = 1;
// เพิ่มคุณสมบัติเพื่อเซฟข้อมูล uid ไปยังการเชื่อมต่อ (uid คือ ไอดีของผู้ใช้ หรือ สิ่งที่แตกต่างเหมือนแท็กของไคลเอนต์)
$worker->uidConnections = array();
// เมื่อมีการส่งข้อมูลจากไคลเอ็นต์
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // ตรวจสอบว่าไคลเอนต์นี้ได้รับการตรวจสอบหรือยัง กล่าวคือการตั้งค่าของ uid หรือเปล่า
    if(!isset($connection->uid))
    {
       // หากไม่ได้รับการตรวจสอบ จะให้บรรทัดแรกเป็น uid (ที่นี่เพื่อการสะดวกตอนทดสอบ เพราะๆไม่ได้ทำการตรวจสอบจริง)
       $connection->uid = $data;
       /* บันทึก uid ไปยังการเชื่อมต่อ เพื่อที่จะสามารถใช้ uid เพื่อค้นหาการเชื่อมต่อ และทำการส่งข้อมูลที่เฉพาะเจาะจง */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // ตรวจสอบเงื่อนไขอื่นๆ สำหรับการส่งข้อมูลที่เฉพาะเจาะจง หรือ การส่งข้อมูลให้ทุกคน
    // ถ้ารูปแบบข้อความคือ uid:message คือการส่ง message ไปยัง uid
    // uid คือ all คือการส่งข้อมูลให้ทุกคน
    list($recv_uid, $message) = explode(':', $data);
    // การส่งข้อมูลให้ทุกคน
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // การส่งข้อมูลไปยัง uid ที่เฉพาะเจาะจง
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// เมื่อไคลเอ็นต์ไม่ได้เชื่อมต่อ
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // เมื่อการเชื่อมต่อหลุดให้ลบการเชื่อมต่อ
        unset($worker->uidConnections[$connection->uid]);
    }
};

// การส่งข้อมูลที่ตรงกับ uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// ทำให้ worker คงทำงาน
Worker::runAll();
```

**การอธิบาย:**

ตัวอย่างข้างต้นสามารถทำการส่งข้อมูลที่ตรงกับ uid แม้ว่าจะเป็นกระบวนการทำงานเดียว แต่ก็สามารถรองรับไคลเอนต์ออนไลน์ถึง 10 หมื่นคนได้

ควรจะทำรายการทำงานเฉพาะว่ามีแค่กระบวนการทำงานเดียว นั่นคือ $worker->count จำเป็นต้องเป็น 1 เท่านั้น หากต้องการรองรับหลายกระบวนการทำงานหรือคลัสเตอร์เซิร์ฟเวอร์ จะต้องใช้คอมโพเนนต์ช่องสัญญาณเพื่อการสื่อสารระหว่างกระบวนการทำงาน การพัฒนาก็ไม่ยากอย่างที่คิด สามารถอ้างอิงได้ที่[ อาศัย Channel โครงสร้างหัวข้อรวม](../components/channel-examples.md)เรื่องการส่งข้อมูลในการทำงานหรือคลัสเตอร์เซิร์ฟเวอร์

**หากต้องการที่จะส่งข้อมูลให้กับไคลเอนต์ในระบบอื่น ๆ สามารถอ้างอิงได้ที่[ การทำการส่งในโปรเจคอื่น ๆ](push-in-other-project.md)**
