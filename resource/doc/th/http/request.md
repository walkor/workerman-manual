# คำอธิบาย
Workerman ได้เสริมระบบบริการ HTTP ตั้งแต่เวอร์ชัน 4.x ขึ้นไป โดยรวมเข้ามาเป็นคลาสของคำขอ (Request) คลาสของการตอบกลับ (Response) และซีชัน (Session) รวมถึง [SSE](SSE.md) หากคุณต้องการใช้บริการ HTTP ของ Workerman แนะนำให้ใช้ Workerman เวอร์ชัน 4.x หรือเวอร์ชันที่มีระดับสูงกว่า

**โปรดทราบว่านี่เป็นการใช้งานของ Workerman เวอร์ชัน 4.x เท่านั้น ไม่สามารถใช้ได้กับ Workerman เวอร์ชัน 3.x**

##  การได้รับ Object ของคำขอ
Object ของคำขอจะถูกเรียกใช้ในฟังก์ชัน onMessage โดยการส่ง Object ของคำขอผ่านพารามิเตอร์ที่สองของฟังก์ชันโดยอัตโนมัติ

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request เป็น Object ของคำขอ ที่นี่ไม่ได้ดำเนินการใด ๆ กับ Object ของคำขอและจะตอบกลับ hello ให้กับเบราว์เซอร์โดยตรง
    $connection->send("hello");
};

// ทำงาน worker
Worker::runAll();
```

เมื่อเบราว์เซอร์เข้าถึง `http://127.0.0.1:8080` จะได้รับการตอบกลับ `hello`

## ได้รับพารามิเตอร์ที่แนบมา
**ได้รับอาร์เรย์ get ทั้งหมด**
```php
$get = $request->get();
```
หากคำขอไม่มีพารามิเตอร์ get จะได้รับอาร์เรย์ว่าง

**ได้รับค่าที่อยู่ในอาร์เรย์ get**
```php
$name = $request->get('name');
```
หากอาร์เรย์ get ไม่มีค่านี้ จะได้ค่า null

คุณยังสามารถส่งค่าเริ่มต้นเข้าไปในพารามิเตอร์ที่สองของเมธอด get หากอาร์เรย์ get ไม่มีค่าที่ต้องการจะได้รับค่าเริ่มต้น ตัวอย่างเช่น:
```php
$name = $request->get('name', 'tom');
```

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->get('name'));
};

// ทำงาน worker
Worker::runAll();
```

เมื่อเบราว์เซอร์เข้าถึง `http://127.0.0.1:8080?name=jerry&age=12` จะได้รับการตอบกลับ `jerry`

## ได้รับพารามิเตอร์ post
**ได้รับอาร์เรย์ post ทั้งหมด**
```php
$post = $request->post();
```
หากคำขอไม่มีการส่งพารามิเตอร์ post จะได้รับอาร์เรย์ว่าง

**ได้รับค่าที่อยู่ในอาร์เรย์ post**
```php
$name = $request->post('name');
```
หากอาร์เรย์ post ไม่มีค่านี้ จะได้ค่า null

เหมือนกับเมธอด get คุณยังสามารถส่งค่าเริ่มต้นเข้าไปในพารามิเตอร์ที่สองของเมธอด post หากอาร์เรย์ post ไม่มีค่าที่ต้องการจะได้รับค่าเริ่มต้น ตัวอย่างเช่น:
```php
$name = $request->post('name', 'tom');
```
**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// ทำงาน worker
Worker::runAll();
```

## ได้รับข้อมูลโพสต์การร้องขอเป็นรูปแบบเดิม
```php
$post = $request->rawBody();
```
ฟังก์ชันนี้ทำงานคล้ายกับการใช้ `file_get_contents("php://input");` ใน `php-fpm` ที่ใช้สำหรับการจับผิดจากการร้องขอแบบเดิมขอไอพีวี

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// ทำงาน worker
Worker::runAll();
```

## ได้รับส่วนหัว
**ได้รับอาร์เรย์ส่วนหัวทั้งหมด**
```php
$headers = $request->header();
```
หากคำขอไม่มีส่วนหัว จะได้รับอาร์เรย์ว่าง โปรดทราบว่าทุก ๆ คีย์จะเป็นตัวพิมพ์เล็ก

**ได้รับค่าที่อยู่ในอาร์เรย์ส่วนหัว**
```php
$host = $request->header('host');
```
หากอาร์เรย์ส่วนหัว ไม่มีค่านี้ จะได้ค่า null โปรดทราบว่าทุก ๆ คีย์จะเป็นตัวพิมพ์เล็ก

เหมือนกับเมธอด get คุณยังสามารถส่งค่าเริ่มต้นเข้าไปในพารามิเตอร์ที่สองของเมธอด header หากอาร์เรย์ส่วนหัว ไม่มีค่าที่ต้องการจะได้รับค่าเริ่มต้น ตัวอย่างเช่น:
```php
$host = $request->header('host', 'localhost');
```

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// ทำงาน worker
Worker::runAll();
```
## รับคุกกี้
**รับอาร์เรย์คุกกี้ทั้งหมด**
```php
$cookies = $request->cookie();
```
หากคำขอไม่มีพารามิเตอร์คุกกี้ จะคืนค่าเป็นอาร์เรย์ว่างๆ

**รับค่าหนึ่งของอาร์เรย์คุกกี้**
```php
$name = $request->cookie('name');
```
หากอาร์เรย์คุกกี้ไม่มีค่านี้ จะคืนค่าเป็น null

เหมือนกับวิธีการที่เราใช้ใน get คุณสามารถส่งพารามิเตอร์ตัวสองของ cookie ไปยังเมทอด ถ้าไม่พบค่าที่เกี่ยวข้องในอาร์เรย์ cookie จะคืนค่ากำหนดเอง เช่น
```php
$name = $request->cookie('name', 'tom');
```
**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// รันเวิร์กเกอร์
Worker::runAll();
```

## รับไฟล์อัปโหลด
**รับอาร์เรย์ไฟล์อัปโหลดทั้งหมด**
```php
$files = $request->file();
```
รูปแบบของไฟล์ที่ได้จะเป็นดังนี้
```php
array (
    'avatar' => array (
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' =>  array (
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
โดยที่:

 - ชื่อเป็นชื่อของไฟล์
 - tmp_name เป็นตำแหน่งของไฟล์ชั่วคราวบนดิสก์
 - ขนาดคือขนาดของไฟล์
 - error เป็น[รหัสข้อผิดพลาด](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - ประเภท เป็นประเภทของไฟล์

**หมายเหตุ:**

 - ขนาดของไฟล์อัปโหลดถูก จำกัด โดย [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md) ค่าเริ่มต้นคือ 10M ซึ่งสามารถแก้ไขได้
 - ไฟล์จะถูกลบโดยอัตโนมัติหลังการขอ
 - หากคำขอไม่มีไฟล์ที่อัปโหลดจะคืนอาร์เรย์ว่างๆ

### รับไฟล์อัปโหลดที่โดดเด่น
```php
$avatar_file = $request->file('avatar');
```
แบบที่ได้จะเป็นดังนี้
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
หากไม่มีไฟล์อัปโหลดหรือไม่พบไฟล์ที่ระบุจะคืนค่าเป็น null

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// รันเวิร์กเกอร์
Worker::runAll();
```
## รับโฮสต์
รับข้อมูลโฮสต์ของคำขอ
```php
$host = $request->host();
```
หากที่อยู่ของคำขอไม่ใช่พอร์ตมาตรฐาน 80 หรือ 443 ข้อมูลโฮสต์อาจจะรวมถึงพอร์ต เช่น `example.com:8080` หากคุณไม่ต้องการพอร์ต คุณสามารถส่งพารามิเตอร์ที่หนึ่งไปกับ `true`

```php
$host = $request->host(true);
```
**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->host());
};

// รันเวิร์กเกอร์
Worker::runAll();
```
เมื่อเบราว์เซอร์เข้าถึง `http://127.0.0.1:8080?name=tom` จะคืนค่าเป็น `127.0.0.1:8080`

## รับเมทอดคำขอ
```php
$method = $request->method();
```
ค่าที่คืนไปอาจเป็น `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// รันเวิร์กเกอร์
Worker::runAll();
```
## รับ URI ของคำขอ
```php
$uri = $request->uri();
```
คืนค่า URI ของคำขอ รวมถึง path และ queryString

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// รันเวิร์กเกอร์
Worker::runAll();
```
เมื่อเบราว์เซอร์เข้าถึง `http://127.0.0.1:8080/user/get.php?uid=10&type=2` จะคืนค่าเป็น `/user/get.php?uid=10&type=2`
## รับเส้นทางการร้องขอ

```php
$path = $request->path();
```
คืนค่าเส้นทางที่ร้องขอมา

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// รัน worker
Worker::runAll();
```
เมื่อเบราว์เซอร์เข้าถึง `http://127.0.0.1:8080/user/get.php?uid=10&type=2` จะคืนค่าเป็น `/user/get.php` 

## รับข้อมูลคิวรี่ของการร้องขอ

```php
$query_string = $request->queryString();
```
คืนค่าส่วนคิวรี่ของการร้องขอ

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// รัน worker
Worker::runAll();
```
เมื่อเบราว์เซอร์เข้าถึง `http://127.0.0.1:8080/user/get.php?uid=10&type=2` จะคืนค่าเป็น `uid=10&type=2` 

## รับเวอร์ชัน HTTP ของการร้องขอ

```php
$version = $request->protocolVersion();
```
คืนค่าเป็นสตริง `1.1` หรือ `1.0`

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// รัน worker
Worker::runAll();
```

## รับเซสชัน ID ของการร้องขอ

```php
$sid = $request->sessionId();
```
คืนค่าเป็นสตริงที่ประกอบไปด้วยตัวอักษรและตัวเลข

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// รัน worker
Worker::runAll();
```
