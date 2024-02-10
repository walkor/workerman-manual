# คอนสตรักเตอร์ __construct

## คำอธิบาย:
```php
Worker::__construct([string $listen , array $context])
```

เริ่มต้นอินสแตนซ์ของ Worker container ซึ่งสามารถตั้งค่าคุณสมบัติและตัวกำหนดระบบสัญญาณ callback สำหรับการทำงานที่เฉพาะเจาะจง


## พารามิเตอร์
#### **`$listen`** (พารามิเตอร์ที่ไม่จำเป็น หากไม่มีการระบุจะหมายถึงไม่ได้รับการร้องขอที่สถานี)
หากมีการตั้งค่าพารามิเตอร์ `$listen` จะทำการติดตาม socket

รูปแบบของ `$listen` คือ <โปรโตคอล>://<ที่อยู่ที่ต้องการติดตาม>
**<โปรโตคอล> อาจจะมีรูปแบบดังต่อไปนี้:** 

tcp: เช่น ```tcp://0.0.0.0:8686```

udp: เช่น ```udp://0.0.0.0:8686```

unix: เช่น ```unix:///tmp/my_file ``` ```(ต้องมี Workerman>=3.2.7)```

http: เช่น ```http://0.0.0.0:80```

websocket: เช่น ```websocket://0.0.0.0:8686```

text: เช่น ```text://0.0.0.0:8686``` ```(text คือโปรโตคอลเอกสารภายในของ Workerman ที่เข้ากันได้กับ telnet ดูรายละเอียดในส่วนของโครงสร้างข้อมูล Text)```

รวมถึงโปรโตคอลที่กำหนดเองอื่น ๆ ดูที่คูมือผู้ใช้สร้างพิเศษสัญญาณการสื่อสาร

**<ที่อยู่ที่ต้องการติดตาม> อาจมีรูปแบบดังต่อไปนี้:** 

หากเป็นช่องเชื่อมต่อยูนิกซ์ ที่อยู่จะเป็นเส้นทางดิสก์ในเครื่องเดียวกัน

ถ้าไม่ใช่ช่องเชื่อมต่อยูนิกซ์ รูปแบบของที่อยู่คือ <ไอพีของเครื่อง>:<หมายเลขพอร์ต>
<ไอพีของเครื่อง> สามารถเป็น```0.0.0.0``` หมายถึงการติดตามทุกและทั้งภายในเครื่อง รวมถึงไอพีภายในและไอพีภายนอกและหยุดวงจรในคอมพิวเตอร์ภายใน 127.0.0.1

<ไอพีของเครื่อง> ถ้าเป็น```127.0.0.1``` หมายถึงกำลังติดตามวงจรในเครื่อง ซึ่งเฉพาะเฉพาะการเข้าถึงจากส่วนนอก

<ไอพีของเครื่อง> ถ้าเป็นไอพีภายใน เช่น```192.168.xx.xx``` หมายถึงเฉพาะการติดตามไอพีภายใน โดยที่ผู้ใช้ภายนอกจะไม่สามารถเข้าถึงได้

ค่าที่ได้รับการตั้งค่าของ<ไอพีของเครื่อง> ที่ไม่ได้อยู่ในส่วนของเครื่องจะไม่สามารถรักษาการติดตาม และเเจ้งเตือนผลการทำงานไม่สามารถกำหนดข้อความข้อความและดำเนินการเบื้องต้นได้

**โปรดทราบ:**  <หมายเลขพอร์ต> ไม่สามารถมีค่ามากกว่า 65535 <หมายเลขพอร์ต> หากมีค่าน้อยกว่า 1024 จะต้องมีการใช้สิทธิ์ root เพื่อติดตาม พอร์ตที่ต้องการติดตามต้องเป็นพอร์ตที่ยังไม่ถูกใช้งานไปและเเถ
าไม่เคยใช้งานไปขึ้นข้อความการติดตาม และเเจ้ดเตือน```Address already in use```ข้อผิดพลาด

#### **`$context`**
เป็นอาเรย์ ใช้สำหรับการส่งตัวเลือกระบบเชื่อมต่อ socket ดู [อตีอัตต์เลือก socket](https://php.net/manual/zh/context.socket.php)
## ตัวอย่าง

Worker ทำหน้าที่เป็นคอนเทนเนอร์ HTTP ที่รอรับและประมวลผลคำขอ HTTP
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send("hello");
};

// รัน worker
Worker::runAll();
```

Worker ทำหน้าที่เป็นคอนเทเนอร์ WebSocket ที่รอรับและประมวลผลคำขอ WebSocket
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// รัน worker
Worker::runAll();
```

Worker ทำหน้าที่เป็นคอนเทนเนอร์ TCP ที่รอรับและประมวลผลคำขอ TCP
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// รัน worker
Worker::runAll();
```

Worker ทำหน้าที่เป็นคอนเทนเนอร์ UDP ที่รอรับและประมวลผลคำขอ UDP
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// รัน worker
Worker::runAll();
```

Worker รับฟังการเชื่อมต่อด้วย domain socket ```(ต้องใช้ Workerman เวอร์ชัน >= 3.2.7)```
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('unix:///tmp/my.sock');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send("hello");
};

// รัน worker
Worker::runAll();
```

Worker ไม่ได้รับฟังการเชื่อมต่อใด ๆ เพื่อใช้ในการประมวลผลงานที่เกี่ยวกับการทำงานตามเวลาที่สม่ำเสมอ
```php
use \Workerman\Worker;
use \Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function($task)
{
    // ทุก 2.5 วินาที จะทำงาน
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// รัน worker
Worker::runAll();
```

Worker รับฟังและประมวลผลพอรโตคอลที่ผู้ใช้กำหนด
```php
├── Protocols              // นี่คือโฟลเดอร์ Protocols ที่จะสร้างขึ้นมา
│   └── MyTextProtocol.php // นี่คือไฟล์พรอโตคอลที่ผู้ใช้กำหนด
├── test.php  // นี่คือสคริปต์ test ที่จะสร้างขึ้นมา
└── Workerman // ไดเรกทอรีโค้ดต้นฉบับของ Workerman โปรดอย่าแก้ไขโค้ดภายใน

1. สร้างโฟลเดอร์ Protocols และสร้างไฟล์พรอโตคอล
Protocols/MyTextProtocol.php (ดูโครงสร้างโฟลเดอร์ด้านบน)

```php
// ขนาดบนเนมสเปซของโปรโตคอลที่ผู้ใช้กำหนดเป็น Protocols
namespace Protocols;
// โปรโตคอลข้อความธรรมดา รูปแบบโปรโตคอลคือ ข้อความ+เส้นใหม่
class MyTextProtocol
{
    // ฟังก์ชั่นแบ่งชุดข้อมูล คืนค่าความยาวของชุดปัจจุบัน
    public static function input($recv_buffer)
    {
        // ค้นหาเส้นใหม่
        $pos = strpos($recv_buffer, "\n");
        // ถ้าไม่พบเส้นใหม่ แสดงว่าไม่ใช่ชุดเต็ม คืนค่า 0 เพื่อรอข้อมูลต่อไป
        if($pos === false)
        {
            return 0;
        }
        // พบเส้นใหม่ คืนค่าความยาวของชุดปัจจุบัน รวมถึงเส้นใหม่
        return $pos+1;
    }

    // เมื่อได้รับชุดข้อมูลเต็มเรียบร้อย จะถูกส่งผ่านการถอดรหัสอัตโนมัติด้วยการถอดรหัส (decode) ซึ่งในที่นี้คือตัดเส้นใหม่ออก
    public static function decode($recv_buffer)
    {
        return trim($recv_buffer);
    }

    // ก่อนส่งข้อมูลไปยังไคลเอนต์จะถูกผ่านการเข้ารหัสอัตโนมัติด้วยการเข้ารหัส (encode) ซึ่งที่นี้เพิ่มเส้นใหม่
    public static function encode($data)
    {
        return $data."\n";
    }
}
```

2. ใช้โปรโตคอล MyTextProtocol เพื่อการรับฟังและประมวลผลคำขอ

ดูโครงสร้างโฟลเดอร์และสร้างไฟล์ test.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// #### MyTextProtocol worker ####
$text_worker = new Worker("MyTextProtocol://0.0.0.0:5678");

/*
 * เมื่อได้รับชุดข้อมูลเต็ม (จบด้วยเส้นใหม่) จะโดยอัตโนมัติเรียก MyTextProtocol::decode('ข้อมูลที่ได้รับ')
 * ผลลัพธ์จะถูกส่งผ่าน $data ไปยัง onMessage callback
 */
$text_worker->onMessage =  function(TcpConnection $connection, $data)
{
    var_dump($data);
    /*
     * ส่งข้อมูลกลับไปยังไคลเอนต์ จะได้รับการเรียกผ่านการเข้ารหัสอัตโนมัติด้วย MyTextProtocol::encode('hello world')
     * และจึงจะถูกส่งไปยังไคลเอนต์
     */
    $connection->send("hello world");
};

// รัน worker ทั้งหมด
Worker::runAll();
```

3. ทดสอบ

เปิดหน้าต่างคำสั่ง ไปยังไดเรกทอรีที่มีไฟล์ test.php และรันคำสั่ง ```php test.php start```
```shell
php test.php start
Workerman[test.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.2.7          PHP version:5.4.37
------------------------ WORKERS -------------------------------
user          worker        listen                         processes status
root          none          myTextProtocol://0.0.0.0:5678   1         [OK]
----------------------------------------------------------------
Press Ctrl-C to quit. Start success.
```

เปิดหน้าต่างคำสั่งอีกหนึ่งตัว โดยใช้ telnet เพื่อทดสอบ (แนะนำให้ใช้ telnet ของระบบปฏิบัติการลินุกซ์)
เมื่อทดสอบบนเครื่องที่เป็นเครื่องเซิฟเวอร์
ใช้คำสั่ง telnet 127.0.0.1 5678
จากนั้นพิมพ์คำว่า hi และกด Enter
คุณจะได้รับข้อมูล hello world\n
```shell
telnet 127.0.0.1 5678
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
hi
hello world
```
