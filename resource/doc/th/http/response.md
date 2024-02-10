# คำอธิบาย

workerman เริ่มเวอร์ชัน 4.x ได้เสริมการสนับสนุนการให้บริการ HTTP มากขึ้น โดยมีการนำเข้าคลาสของคำขอ (Request), คลาสของการตอบ (Response), คลาสของเซสชัน (Session) และ [SSE](SSE.md) หากคุณต้องการใช้บริการ HTTP ของ workerman ขอแนะนำให้ใช้ workerman เวอร์ชัน 4.x หรือเวอร์ชันที่สูงกว่า

**โปรดทราบว่าตัวอย่างทั้งหมดนี้เป็นการใช้งานของ workerman เวอร์ชัน 4.x ซึ่งไม่รองรับเวอร์ชัน 3.x**


# คำเตือน

- ยกเว้นว่าจะเป็นการส่งคังคื้นหรือตอบสนองตัวเลขการส่งเดี่ยวหรือ SSE แล้วจะไม่อนุญาตให้ส่งตอบสนองหลายครั้งในคำขอเดียว ก็คือไม่อนุญาตให้เรียก `$connection->send()` หลายครั้งในคำขอเดียว
- ทุกคำขอต้องเรียก `$connection->send()` อย่างน้อยหนึ่งครั้งเพื่อส่งตอบสนอง ไม่งั้นก็จะมีลูกค้ารออยู่ตลอดเวลา

## การตอบสนองโดยสั่งแม่

เมื่อไม่ต้องการเปลี่ยนการเดินทางของ HTTP (ค่าเริ่มต้นคือ 200) หรือกำหนด header, คุ๊กกี้เอง สามารถส่งสตริงตอบสมองกลับต่างรายการได้

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // ส่งตรงถึงลูกค้าข้อความ "this is body"
    $connection->send("this is body");
};

// รัน worker
Worker::runAll();
```

## การเปลี่ยนแปลงการเดินทาง

เมื่อต้องการกำหนดการเดินทางแบบเอง, เปลี่ยน header, คุ๊กกี้ จะต้องใช้คลาสการตอบสนอง `Workerman\Protocols\Http\Response` เช่นตัวอย่างล่างที่เราส่งสถานะ 404 เมื่อเรียกรายการเป็น '/404', และกระเบียงส่วนตัวคือ '<h1>ขอโทษ ไม่พบไฟล์</h1>'

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>ขอโทษ ไม่พบไฟล์</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// รัน worker
Worker::runAll();
```
เมื่อคลาส `Response` ถูกสร้างขึ้น และต้องการเปลี่ยนสถานะสามารถใช้วิธีด้านล่าง
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## ส่ง header

เช่นกัน, เพื่อส่ง headerต้องใช้คลาสการตอบสนอง `Workerman\Protocols\Http\Response`

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Header Value'
    ], 'this is body');
    $connection->send($response);
};

// รัน worker
Worker::runAll();
```
เมื่อคลาส `Response` ถูกสร้างขึ้นและต้องการเพิ่มหรือเปลี่ยนหัวเรื่องใช้วิธีด้านล่าง
```php
$response = new Response(200);
// เพิ่มหรือเปลี่ยนหัวเรื่อง
$response->header('Content-Type', 'text/html');
// เพิ่มหรือเปลี่ยนหลายหัวเรื่อง
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Header Value'
]);
$connection->send($response);
```

## การเปลี่ยนเส้นทาง

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```


## ส่งคุ๊กกี้

เช่นกัน, ส่งคุ๊กกี้ต้องใช้คลาสการตอบสนอง `Workerman\Protocols\Http\Response`

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'this is body');
    $response->cookie('name', 'tom');
    $connection->send($response);
};

// รัน worker
Worker::runAll();
```
## ส่งไฟล์
เช่นเดียวกับนั้น การส่งไฟล์ต้องใช้คลาสการตอบกลับ `Workerman\Protocols\Http\Response` 

ใช้วิธีดังนี้เพื่อส่งไฟล์
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
 - workerman รองรับการส่งไฟล์ขนาดใหญ่
 - สำหรับไฟล์ขนาดใหญ่ (มากกว่า 2M) workerman จะไม่อ่านไฟล์ทั้งหมดเข้าสู่หน่วยความจำที่ด้วยตัวครั้งเดียว แต่จะอ่านและส่งไฟล์ชิ้นเป็นชิ้นในเวลาที่เหมาะสม
 - workerman จะปรับปรุงความเร็วในการอ่านและส่งไฟล์ตามความเร็วในการรับข้อมูลของผู้ใช้ เพื่อให้การส่งไฟล์เร็วที่สุด พร้อมลดการใช้หน่วยความจำลง
 - การส่งข้อมูลไม่ block และไม่มีผลต่อการประมวลผลคำขออื่น ๆ
 - ขณะส่งไฟล์จะถูกเพิ่ม `Last-Modified` หัวข้ออัตโนมัติเพื่อให้เซิร์ฟเวอร์ตรวจสอบว่าจะส่งการตอบกลับ 304 เพื่อประหยัดการถ่ายทอดไฟล์และเพิ่มประสิทธิภาพ
 - ไฟล์ที่ส่งจะถูกส่งอัตโนมัติพร้อมกับหัวข้อ `Content-Type` ที่เหมาะสมส่งให้กับเบราว์เซอร์
 - หากไฟล์ไม่มีอยู่ จะถูกเปลี่ยนเป็นการตอบกลับ 404 โดยอัตโนมัติ

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/your/path/of/file';
    // ตรวจสอบไฟล์ว่าเปลี่ยนแปลงหรือไม่โดยใช้ if-modified-since หัวข้อ
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // ถ้าไฟล์ไม่ถูกเปลี่ยนแปลง จะส่งการตอบกลับ 304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // ถ้าไฟล์เปลี่ยนแปลงหรือไม่มี if-modified-since หัวข้อ จะส่งไฟล์
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// รันเวิร์กเกอร์
Worker::runAll();
```


## ส่งข้อมูล http chunk
 - จำเป็นต้องส่งการตอบกลับ Response ที่มีหัวข้อ `Transfer-Encoding: chunked` ให้กับผู้ใช้ก่อน
 - ส่งข้อมูล chunk ต่อไปโดยใช้คลาส `Workerman\Protocols\Http\Chunk`
 - สุดท้ายจะต้องส่ง chunk ว่างเพื่อจบการตอบกลับ

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // ส่งการตอบกลับ Response ที่มีหัวข้อ `Transfer-Encoding: chunked` ให้กับผู้ใช้ก่อน
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // ส่งข้อมูล chunk ต่อไปโดยใช้คลาส Workerman\Protocols\Http\Chunk
    $connection->send(new Chunk('ข้อมูลชิ้นที่หนึ่ง'));
    $connection->send(new Chunk('ข้อมูลชิ้นที่สอง'));
    $connection->send(new Chunk('ข้อมูลชิ้นที่สาม'));
   // สุดท้ายจะต้องส่ง chunk ว่างเพื่อจบการตอบกลับ
    $connection->send(new Chunk(''));
};

// รันเวิร์กเกอร์
Worker::runAll();
```
