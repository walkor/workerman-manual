# คำอธิบาย

ตั้งแต่เวอร์ชัน 4.x ของ workerman ได้เสริมการสนับสนุนบริการ HTTP โดยในเวอร์ชันนี้มีการนำเข้าคลาสของ request, response, session และ [SSE](SSE.md) ถ้าคุณต้องการใช้บริการ HTTP ของ workerman แนะนำให้ใช้เวอร์ชัน 4.x หรือเวอร์ชันที่สูงกว่า

**โปรดระวังว่านี้เป็นการใช้งานของ workerman เวอร์ชัน 4.x เท่านั้น ไม่สามารถใช้งานได้กับ workerman เวอร์ชัน 3.x**


# รับออบเจกต์เซสชัน
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// ทำงาน worker
Worker::runAll();
```
**ข้อควรระวัง**
- ต้องการจัดการเซสชันก่อนเรียก `$connection->send()` 
- เมื่อวัตถุเซสชันถูกทำลาย จะบันทึกการเปลี่ยนแปลงโดยอัตโนมัติ ดังนั้นอย่าเก็บวัตถุที่ได้จาก `$request->session()` ไว้ในอาร์เรย์ทั่วไปหรือเป็นฟิลด์ของคลาส ซึ่งอาจทำให้เซสชันไม่สามารถบันทึกได้
- ค่าเริ่มต้นของเซสชันจะถูกบันทึกในไฟล์แฟ้ม หากต้องการประสิทธิภาพที่ดีขึ้นแนะนำให้ใช้ Redis

## รับข้อมูลเซสชันทั้งหมด
```php
$session = $request->session();
$all = $session->all();
```
คืนค่าเป็นอาร์เรย์ หากไม่มีข้อมูลเซสชันใดๆ จะคืนค่าเป็นอาร์เรย์ว่าง

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// ทำงาน worker
Worker::runAll();
```

## รับค่าในเซสชัน
```php
$session = $request->session();
$name = $session->get('name');
```
หากข้อมูลไม่มีจะคืนค่าเป็น null

คุณสามารถส่งค่าเริ่มต้นเป็นอาร์กิวเมนต์สองหากค้นหาไม่เจอค่าที่ต้องการจะคืนค่าเริ่มต้น ตัวอย่างเช่น:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// ทำงาน worker
Worker::runAll();
```


## บันทึกเซสชัน
เพื่อบันทึกข้อมูลในเซสชันใช้เมทอด set
```php
$session = $request->session();
$session->set('name', 'tom');
```
set ไม่มีการคืนค่า และเซสชันจะถูกบันทึกอัตโนมัติเมื่อวัตถุเซสชันถูกทำลาย

เมื่อต้องการบันทึกค่ามากกว่าหนึ่งค่าใช้เมทอด put
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
เช่นเดียวกัน put ไม่มีการคืนค่า

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// ทำงาน worker
Worker::runAll();
```


## ลบข้อมูลเซสชัน
เพื่อลบข้อมูลเซสชันใช้เมทอด `forget`
```php
$session = $request->session();
// ลบข้อมูล
$session->forget('name');
// ลบหลายข้อมูล
$session->forget(['name', 'age']);
```

นอกจากนี้ยังมีเมทอด delete ซึ่งต่างจาก forget โดย delete สามารถลบข้อมูลเพียงหนึ่งอย่าง
```php
$session = $request->session();
// เหมือนกับ $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// ทำงาน worker
Worker::runAll();
```
## รับค่าและลบค่า session กลายเป็น
```php
$session = $request->session();
$name = $session->pull('name');
```
ผลเท่ากับรหัสด้านล่าง
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
ถ้า session ที่สอดคต่านั้นไม่มีอยู่ จะคืนค่าเป็น null

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->session()->pull('name'));
};

// รัน worker
Worker::runAll();
```

## ลบข้อมูล session ทั้งหมด
```php
$request->session()->flush();
```
ไม่มีค่าที่คืน, เมื่อ object session ถูกทำลาย session จะถูกลบโดยอัตโนมัติ

**ตัวอย่าง**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->flush();
    $connection->send('ok');
};

// รัน worker
Worker::runAll();
```

## ตรวจสอบว่าข้อมูล session ที่สอดคต่านั้นมีหรือไม่
```php
$session = $request->session();
$has = $session->has('name');
```
เมื่อ session ที่สอดคต่านั้นไม่มีหรือค่าของ session ที่สอดคต่านั้นเป็นค่า null จะได้รับค่าเป็น false, มิฉะนั้นจะได้รับค่าเป็น true

```
$session = $request->session();
$has = $session->exists('name');
```
การจะตรวจสอบข้อมูล session ว่ามีหรือไม่มี เช่นกัน คือจะต่างกันตรงที่เมื่อค่าของ session ที่สอดคต่านั้นเท่ากับ null จะได้รับค่าเป็น true ด้วย
