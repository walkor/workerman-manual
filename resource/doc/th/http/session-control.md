# คำอธิบาย
Workerman ได้เสริมความสามารถในการให้บริการ HTTP ตั้งแต่เวอร์ชัน 4.x ขึ้นไป โดยใช้ชั้นการขอร้อง ชั้นการตอบรับ คลาสเซสชันและ [SSE](SSE.md) ได้นำเข้ามา หากคุณต้องการใช้บริการ HTTP ขอแนะนำให้ใช้ Workerman เวอร์ชัน 4.x หรือเวอร์ชันที่สูงขึ้น

**โปรดทราบว่าตัวอย่างทั้งหมดนี้ใช้กับ Workerman เวอร์ชัน 4.x และไม่สามารถใช้กับ Workerman เวอร์ชัน 3.x ได้**

## เปลี่ยนเครื่องเก็บข้อมูลเซสชัน
Workerman มีการเตรียมเครื่องเก็บข้อมูลเซสชันแบบไฟล์ และเครื่องเก็บข้อมูลเซสชันแบบรีดิส ค่าเริ่มต้นคือการเก็บข้อมูลเซสชันแบบไฟล์ หากต้องการเปลี่ยนเป็นเครื่องเก็บข้อมูลเซสชันแบบรีดิส โปรดดูตัวอย่างโค้ดด้านล่าง

```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// กำหนดค่ารีดิส
$config = [
    'host'     => '127.0.0.1', // พารามิเตอร์ที่จำเป็น
    'port'     => 6379,        // พารามิเตอร์ที่จำเป็น
    'timeout'  => 2,           // พารามิเตอร์ที่เลือกได้
    'auth'     => '******',    // พารามิเตอร์ที่เลือกได้
    'database' => 1,           // พารามิเตอร์ที่เลือกได้
    'prefix'   => 'session_'   // พารามิเตอร์ที่เลือกได้
];
// ใช้ Workerman\Protocols\Http\Session::handlerClass เพื่อเปลี่ยนคลาสตั้งต้นของเซสชัน
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## กำหนดตำแหน่งเก็บข้อมูลเซสชัน
เมื่อใช้เครื่องเก็บข้อมูลเซสชันแบบค่าเริ่มต้น ข้อมูลเซสชันจะถูกเก็บไว้บนดิสก์ที่ตำแหน่งเริ่มต้นจาก `session_save_path()` คุณสามารถเปลี่ยนตำแหน่งเก็บข้อมูลได้ดังนี้ 

```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// กำหนดตำแหน่งเก็บข้อมูลของเซสชันไฟล์
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// รันเวิร์คเกอร์
Worker::runAll();
```

## การทำความสะอาดข้อมูลเซสชัน
เมื่อใช้เครื่องเก็บข้อมูลเซสชันแบบค่าเริ่มต้นบนดิสก์ จะมีไฟล์เซสชันหลายไฟล์อยู่บนดิสก์ เวิร์คเกอร์ จะทำความสะอาดไฟล์เซสชันที่เก่าออก โดยใช้ค่าที่ตั้งไว้ใน php.ini `session.gc_probability` `session.gc_divisor` `session.gc_maxlifetime` สามารถอ่านรายละเอียดเพิ่มเติมเกี่ยวกับค่าที่ตั้งใน php.ini ได้ที่ [คู่มือ php](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability)
## เปลี่ยนการทำงานของการจัดเก็บ
นอกเหนือจากการจัดเก็บเซสชันในรูปแบบไฟล์และ redis session ที่มีอยู่ใน workerman คุณสามารถเพิ่มการจัดเก็บเซสชันใหม่โดยใช้อินเทอร์เฟซ [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) เช่นการจัดเก็บเซสชันในรูปแบบ mangoDb, MySQL เป็นต้น

**ขั้นตอนการเพิ่มการจัดเก็บเซสชันใหม่**
1. ปรับปรุงอินเทอร์เฟซ [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) 
2. ใช้เมธอด `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` เพื่อแทนที่อินเทอร์เฟซหลักของ SessionHandler

**ปรับปรุงอินเทอร์เฟซ SessionHandlerInterface**

การสร้างการจัดเก็บเซสชันที่กำหนดเองต้องปรับปรุงอินเทอร์เฟซ SessionHandlerInterface ซึ่งมีเมธอดดังนี้:
```php
SessionHandlerInterface {
    /* Methods */
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**คำอธิบาย SessionHandlerInterface**
 - เมธอด read ใช้สำหรับอ่านข้อมูลเซสชันทั้งหมดที่เก็บไว้ตาม session_id โปรดอย่าทำการตัดสตริงข้อมูล เนื่องจาก framework จะดำเนินการอัตโนมัติ
 - เมธอด write ใช้สำหรับเขียนข้อมูลเซสชันที่เก็บไว้ตาม session_id โปรดอย่าทำการจัดรูปแบบข้อมูล เนื่องจาก framework จะดำเนินการอัตโนมัติ
 - เมธอด destroy ใช้สำหรับทำลายข้อมูลเซสชันตาม session_id
 - เมธอด gc ใช้สำหรับลบข้อมูลเซสชันที่หมดอายุ การจัดเก็บควรลบข้อมูลเซสชันทั้งหมดที่มีเวลาการแก้ไขล่าสุดมากกว่า maxlifetime
 - เมธอด close ไม่จำเป็นต้องดำเนินการใด ๆ เพียงแค่ส่งค่าคืนเป็น true เท่านั้น
 - เมธอด open ไม่จำเป็นต้องดำเนินการใด ๆ เพียงแค่ส่งค่าคืนเป็น true เท่านั้น

**การแทนที่ดราย์เวอร์ทั้งหลัง**

หลังจากการปรับปรุงอินเทอร์เฟซ SessionHandlerInterface เสร็จสิ้น ให้ใช้เมธอดต่อไปนี้เพื่อแทนที่ดราย์เวอร์ทั้งหลังของเซสชัน

```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name คือชื่อคลาสของ SessionHandler ที่ปรับปรุงอินเทอร์เฟซ SessionHandlerInterface เอาไว้ หากมีเนมสเปซต้องระบุเนมสเปซเต็ม
 - $config คือพารามิเตอร์ของคอนสตรัคเตอร์ของคลาส SessionHandler

**การปรับปรุงโดยเฉพาะ**

*โปรดทราบว่าคลาส MySessionHandler นี้ใช้เพียงเพื่ออธิบายกระบวนการการเปลี่ยนการทำงานของการจัดเก็บเซสชันเท่านั้น คลาส MySessionHandler ไม่สามารถใช้ในสภาพแวดล้อมการใช้งานจริงได้*

```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

// สมมติว่าคลาส SessionHandler ที่ปรับปรุงอินเทอร์เฟซใหม่ต้องการการกำหนดค่าบางอย่าง
$config = ['host' => 'localhost'];
// ใช้ Workerman\Protocols\Http\Session::handlerClass($class_name, $config) เพื่อเปลี่ยนการทำงานของการจัดเก็บเซสชัน
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
