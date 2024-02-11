# กระบวนการพื้นฐาน
(เรียกเว็บเซอร์วิสแชตห้องบทเว็บซ็อกเก็ตอย่างง่ายเป็นตัวอย่าง)

#### 1. สร้างโฟลเดอร์โปรเจกต์ที่ที่ที่
เช่น SimpleChat/
เข้าไปในโฟลเดอร์และเรียกใช้ `composer require workerman/workerman` 

#### 2. นำเข้า `vendor/autoload.php` (ตัวจดอจาก composer)
สร้าง start.php และนำเข้า `vendor/autoload.php`
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';
```

#### 3. เลือกโปรโตคอล
ที่นี่เราเลือกใช้โปรโตคอลข้อความ (โปรโตคอลที่ Workerman กำหนดเอง รูปแบบเป็นข้อความ + ขึ้นบรรทัดใหม่)

(ในปัจจุบัน Workerman รองรับ HTTP, Websocket, โปรโตคอลข้อความ หากต้องการใช้โปรโตคอลอื่น ๆ โปรดอ้างถึงบทที่โปรโตคอลเพื่อพัฒนาโปรโตคอลของคุณเอง)

#### 4. สร้างสคริปต์เริ่มต้นตามความต้องการ
ตัวอย่างดังต่อไปนี้เป็นไฟล์เข้าสู่ห้องแชตง่าย
SimpleChat/start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$global_uid = 0;

// เมื่อไคลเอนต์เชื่อมต่อเข้ามาจะมีการจัดสรร uid และบันทึกการเชื่อมต่อและแจ้งให้ทราบทุกไคลเอนต์
function handle_connection($connection)
{
    global $text_worker, $global_uid;
    // มอบหมาย uid สำหรับการเชื่อมต่อนี้
    $connection->uid = ++$global_uid;
}

// เมื่อไคลเอนต์ส่งข้อความมา จะถูกส่งต่อให้ทุกคน
function handle_message(TcpConnection $connection, $data)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] พูดว่า: $data");
    }
}

// เมื่อไคลเอนต์ตัดการเชื่อมต่อ จะประกาศไปยังทุกไคลเอนต์
function handle_close($connection)
{
    global $text_worker;
    foreach($text_worker->connections as $conn)
    {
        $conn->send("user[{$connection->uid}] ออกจากระบบ");
    }
}

// สร้าง Worker ของโปรโตคอลข้อความ และฟังก์ชัน handle ด้วย 2347
$text_worker = new Worker("text://0.0.0.0:2347");

// เริ่มต้นพร็อเซสเพียว 1 เพื่องี้สะดวกในการสื่อสารข้อมูลระหว่างไคลเอนต์
$text_worker->count = 1;

$text_worker->onConnect = 'handle_connection';
$text_worker->onMessage = 'handle_message';
$text_worker->onClose = 'handle_close';

Worker::runAll();

```

#### 5. ทดสอบ
โปรโตคอลข้อความสามารถทดสอบด้วยคำสั่ง telnet
```shell
telnet 127.0.0.1 2347
```
