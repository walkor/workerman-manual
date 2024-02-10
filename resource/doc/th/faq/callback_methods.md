# วิธีการเรียกกลับใน PHP

การใช้ฟังก์ชันที่ไม่มีชื่อเพื่อเรียกใช้เป็นวิธีที่สะดวกที่สุดใน PHP แต่นอกเหนือจากวิธีการเรียกกลับแบบฟังก์ชันที่ไม่มีชื่อนั้น ยังมีวิธีการเรียกกลับแบบอื่น ๆ อีกหลายวิธี ดังตัวอย่างต่อไปนี้

## 1. การเรียกกลับแบบฟังก์ชันที่ไม่มีชื่อ
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// การเรียกกลับแบบฟังก์ชันที่ไม่มีชื่อ
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // ส่งข้อความ 'hello world' ไปยังเบราว์เซอร์
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. การเรียกกลับแบบฟังก์ชันทั่กษา
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// การเรียกกลับแบบฟังก์ชันทั่กษา
$http_worker->onMessage = 'on_message';

// ฟังก์ชันทั่กษา
function on_message(TcpConnection $connection, Request $request)
{
    // ส่งข้อความ 'hello world' ไปยังเบราว์เซอร์
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. เมธอดคลาสเป็นการเรียกกลับ
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```

สคริปต์การเริ่มต้น start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// โหลด MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// สร้างออบเจ็กต์
$my_object = new MyClass();

// เรียกใช้เมทอดของคลาส
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

โปรดทราบ: 
โครงสร้างโค้ดด้านบนไม่อนุญาตให้สร้างทรัพยากร (การเชื่อมต่อ MySQL, การเชื่อมต่อ Redis, การเชื่อมต่อ Memcache เป็นต้น) ในคอนสตรักเตอร์ เนื่องจาก ```$my_object = new MyClass();``` รันที่กระบวนการหลัก ตัวอย่างเช่น เมื่อเสร็จสิ้นการเตรียม MySQL จะได้รับการสืบทอดจากระบบย่อยทุกระบบสามารถดำเนินการเชื่อมต่อฐานข้อมูลนี้แต่ทั้งหมดการเชื่อมต่อนี้อยู่ที่เซิร์ฟเวอร์ MySQL และอาจมีข้อผิดพลาดที่ไม่คาดคิด เช่น ข้อผิดพลาด "mysql gone away" ข้อผิดพลาด

โครงสร้างโค้ดด้านบน หากต้องการสร้างทรัพยากรในคอนสตรักเตอร์ของคลาส สามารถนำมาใช้ในรูปแบบต่อไปนี้
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // สมมติว่าคลาสการเชื่อมต่อฐานข้อมูลคือ MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
สคริปต์การเริ่มต้น start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// เตรียมคลาสใน onWorkerStart
$worker->onWorkerStart = function($worker) {
    // โหลด MyClass
    require_once __DIR__.'/MyClass.php';
    
    // สร้างออบเจ็กต์
    $my_object = new MyClass();

    // เรียกใช้เมทอดของคลาส
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```
โครงสร้างโค้ดด้านบน เมื่อ onWorkerStart รัน ก็จะกลายเป็นระบบย่อย ทุกระบบย่อยกำลังสร้างการเชื่อมต่อ MySQL ของตัวเอง ดังนั้นจึงไม่มีปัญหาในการใช้การเชื่อมต่อร่วมกัน สิ่งที่ยังดีอีกคือ การรองรับโค้ดธุรกิจรีโลด ด้วยรูปแบบนี้เมื่อคุณเปลี่ยนแปลง MyClass.php ระบบย่อยก็จะโหลดเพียงอย่างเดียวซึ่งทำให้มากคือมันสามารถใช้งานได้ทันที
## 4. การใช้เมทอดสแตติกเป็นฟังก์ชันคอลล์แบ็ก

คลาสสแตติก MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```
สคริปต์เริ่มต้น start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// โหลด MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// เรียกใช้เมทอดสแตติกของคลาส
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// หากคลาสมีเนมสเปซ การเขียนโค้ดจะเป็นดังนี้
// $worker->onWorkerStart = array('your\namesapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('your\namesapce\MyClass', 'onConnect');
// $worker->onMessage     = array('your\namesapce\MyClass', 'onMessage');
// $worker->onClose       = array('your\namesapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('your\namesapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

โปรดทราบ: ตามกฎการทำงานของ PHP หากไม่มีการเรียกใช้ new จะไม่มีการเรียกใช้ฟังก์ชันคอนสตรักเตอร์ นอกจากนี้เมทอดของคลาสสแตติกไม่อนุญาตให้ใช้ ```$this``` ภายในโค้ด
