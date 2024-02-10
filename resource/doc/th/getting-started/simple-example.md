# ตัวอย่างการพัฒนาที่ง่าย

## การติดตั้ง

**ติดตั้ง Workerman**
ในไดเรกทอรีว่าง ๆ รัน
`composer require workerman/workerman`

## ตัวอย่าง 1: ให้บริการเว็บผ่านโปรโตคอล HTTP
**สร้างไฟล์ start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// สร้าง Worker เพื่อฟังการที่พอร์ต 2345 โดยใช้โปรโตคอล http
$http_worker = new Worker("http://0.0.0.0:2345");

// เปิดให้บริการด้วยดีแบบสามารถบริการพร้อมกับ 4 กระบอกาการ
$http_worker->count = 4;

// เมื่อได้รับข้อมูลจากเบราว์เซอร์ให้ส่งข้อความ 'hello world' กลับไป
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // ส่ง 'hello world' กลับไปยังเบราว์เซอร์
    $connection->send('hello world');
};

// รัน Worker
Worker::runAll();
```

**รันคำสั่งจาก command line (สำหรับผู้ใช้ Windows ให้ใช้ [cmd command line](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn) ตามเดียวกัน)**
```shell
php start.php start

```

**การทดสอบ**

สมมติว่า IP ของเซิร์ฟเวอร์คือ 127.0.0.1

ในเบราว์เซอร์ให้เข้าถึง URL http://127.0.0.1:2345

**หมายเหตุ:**

1. ถ้าไม่สามารถเข้าถึงได้ ให้ดูที่ [เหตุผลที่เกิดจากการเชื่อมต่อล้มเหลวของไคลเอ็นต์](../faq/client-connect-fail.md) เพื่อตรวจสอบ
2. เซิร์ฟเวอร์เป็นโปรโตคอล http เท่านั้น ไม่สามารถใช้โปรโตคอล websocket หรือโปรโตคอลอื่นได้โดยตรง

## ตัวอย่างที่ 2: ให้บริการผ่านโปรโตคอล WebSocket
**สร้างไฟล์ ws_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// คำสั่ง: ต่างจากตัวอย่างก่อนหน้านี้ เราจะใช้โปรโตคอล websocket ที่นี่
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// เปิดให้บริการด้วยดีแบบสามารถบริการพร้อมกับ 4 กระบอกาการ
$ws_worker->count = 4;

// เมื่อได้รับข้อมูลจากไคลเอ็นต์ให้ส่งข้อความ 'hello $data' กลับไป
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // ส่ง 'hello $data' กลับไปยังไคลเอ็นต์
    $connection->send('hello ' . $data);
};

// รัน Worker
Worker::runAll();
```

**รันคำสั่งจาก command line**
```shell
php ws_test.php start

```

**การทดสอบ**

เปิดเบราว์เซอร์ Chrome แล้วกด F12 เพื่อเปิดคอนโซลดบักการควบคุม ในส่วน Console พิมพ์เข้าไป (หรือวางโค้ดด้านล่างไปในหน้าเว็บ HTML เข้าใช้ JavaScript)

```javascript
// สมมติว่า IP ของเซิร์ฟเวอร์คือ 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("เชื่อมต่อสำเร็จ");
    ws.send('tom');
    alert("ส่งข้อความ 'tom' ไปยังเซิร์ฟเวอร์");
};
ws.onmessage = function(e) {
    alert("ได้รับข้อความจากเซิร์ฟเวอร์: " + e.data);
};
``` 

**หมายเหตุ:**

1. ถ้าไม่สามารถเข้าถึงได้ ให้ดูที่ [ข้อผิดพลาดที่พบบ่อยในคู่มือ-การเชื่อมต่อล้มเหลวของไคลเอ็นต์](../faq/client-connect-fail.md) เพื่อตรวจสอบ
2. เซิร์ฟเวอร์เป็นโปรโตคอล websocket เท่านั้น ไม่สามารถใช้โปรโตคอล HTTP หรือโปรโตคอลอื่นได้โดยตรง

## ตัวอย่างที่ 3: การใช้ TCP โดยตรงในการส่งข้อมูล
**สร้างไฟล์ tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// สร้าง Worker ที่ฟังการที่พอร์ต 2347 โดยไม่ใช้โปรโตคอลของชั้นประยุกต์ใด ๆ
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// เปิดให้บริการด้วยดีแบบสามารถบริการพร้อมกับ 4 กระบอกาการ
$tcp_worker->count = 4;

// เมื่อไคลเอ็นต์ส่งข้อมูลมา
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // ส่ง 'hello $data' กลับไปยังไคลเอ็นต์
    $connection->send('hello ' . $data);
};

// รัน Worker
Worker::runAll();
```

**รันคำสั่งจาก command line**

```shell
php tcp_test.php start

```

**การทดสอบ: รันคำสั่งจาก command line**

(ต่อไปนี้คือผลลัพธ์จาก command line บน Linux แต่ละระบบอาจมีความแตกต่าง)
```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```


**หมายเหตุ:**

1. ถ้าไม่สามารถเข้าถึงได้ ให้ดูที่ [ข้อผิดพลาดที่พบบ่อยในคู่มือ-การเชื่อมต่อล้มเหลวของไคลเอ็นต์](../faq/client-connect-fail.md) เพื่อตรวจสอบ
2. เซิร์ฟเวอร์เป็นโปรโตคอล tcp ดั้งย่อ ไม่สามารถใช้โปรโตคอล websocket หรือโปรโตคอล http หรือโปรโตคอลอื่นได้โดยตรง
