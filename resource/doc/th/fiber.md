Workerman 5.0.0 เริ่มรองรับ [Fiber โคไรน์](https://www.php.net/manual/zh/language.fibers.php)

> **โปรดทราบ**
> Fiber ต้องการ PHP>=8.1 และต้องติดตั้ง `composer require revolt/event-loop ^1.0.0`

### แนะนำ

Fiber เป็นโคไรน์ภายในของ PHP ที่สามารถหยุดโค้ด PHP และดำเนินการต่อเมื่อมีความจำเป็น มันมีประโยชน์มากในการเขียนโค้ดแบบซิงโครนัสสำหรับโค้ดการทำงานแบบไม่บล็อค ซึ่งเพิ่มความสามารถในการดูแลรักษาโค้ดได้อย่างมาก

### ตัวอย่าง
ด้านล่างเปรียบเทียบการเขียนโค้ดโคไรน์กับการใช้คอลแบคแบบไม่บล็อค

**การใช้คอลแบคแบบไม่บล็อค**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // ส่งคำขอไปยังอินเทอร์เฟซ HTTP
    $http->get('http://example.com/', function ($response) use ($connection) {
        // ล่าช้า 1 วินาทีก่อนส่งออก
        Timer::add(1, function() use ($connection, $response) {
            // ส่งข้อมูลไปยังเบราว์เซอร์
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**การใช้โคไรน์**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // เรียกใช้งานอินเทอร์เฟซ HTTP
    $response = $http->get('http://example.com/');
    // ล่าช้า 1 วินาที
    Timer::sleep(1);
    // ส่งข้อมูล
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **โปรดทราบ**
> โค้ดด้านบนต้องติดตั้ง composer require workerman/http-client ^2.0.0

ทั้งสองการเขียนโค้ดนี้เป็นการดำเนินการแบบไม่บล็อค ดำเนินการได้ดีทั้งหมดและมีประสิทธิภาพ แต่การใช้โคไรน์ทำให้การอ่านโค้ดได้ง่ายและการดูแลรักษาโค้ดอีกด้วย

### ข้อควรระวังเกี่ยวกับ Fiber
* Fiber โคไรน์ไม่รองรับการทำให้ Pdo, Redis, และฟังก์ชันทางด้านภายในของ PHP เป็นโคไรน์ หมายความว่าการใช้ส่่วนขยายเหล่านี้และฟังก์ชันยังคงเป็นการเรียกที่บล็อค
* Fiber โคไรน์ไคลเอนต์ที่ใช้ได้ปัจจุบันประกอบด้วย [workerman/http-client](../components/workerman-http-client.md) และ [workerman/redis](../components/workerman-redis.md)

# โครงสร้างข้อมูล Swoole
workerman v5 เสนอการรองรับ Swoole ในฐานข้อมูลด้วย

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// ตรงนี้จำเป็นต้องตั้งค่า Swoole เป็นฐานข้อมูลเหตุการณ์
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```

**เกริ่นนำ**
* แนะนำให้ใช้ swoole5.0 หรือเวอร์ชั่นที่สูงขึ้นในภายหลัง
* การเปลี่ยน Swoole เป็นฐานข้อมูลเหตุการณ์จะทำให้ workerman รองรับโครงสร้างข้อมูล Swoole
* การใช้ Swoole เป็นฐานข้อมูลเหตุการณ์สามารถทำได้โดยไม่ต้องติดตั้งส่วนขยายแต่ไม่รองรับการทำโครงสร้างแบบหนึ่งกด หมายความว่าการเรียกใช้ Pdo, Redis, การอ่าน/เขียนไฟล์ของ PHP ภายในเป็นการเรียกใช้แบบบล็อค
* หากต้องการเปิดโครงสร้างแบบหนึ่งกด จำเป็นต้องเรียกใช้  `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

สำหรับข้อมูลเพิ่มเติม โปรดดูที่ [Swoole handbook](https://wiki.swoole.com/)

สำหรับข้อมูลเพิ่มเติม โปรดดูที่ [การขับเคลื่อนด้วยเหตุการณ์](appendices/event.md)

# เกี่ยวกับโครงสร้างแบบหนึ่งกด
ก่อนอื่นไม่จำเป็นที่จะควรที่จะไม่ไว้วางใจโครงสร้างแบบหนึ่งกด ตอนที่ฐานข้อมูล, Redis และการเขียนแฟ้ม PHP อยู่ในเน็ตเวิร์ก การดำเนินงานด้วยกระบวนการอินพุตที่บล็อคจำเป็นอย่างมากเป็นเวลามากกว่าการทำโครงสร้างหนึ่งกด จาก[ข้อมูลการทดสอบซึ่งหลายปีที่มา](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db) ดูว่าการเรียกข้อมูลฐานข้อมูลใน Workerman แบบบล็อคสามารถทำงานได้ดีกว่าการใช้ตัวกลุ่มเชื่อมต่อฐานข้อมูลแบบ Swoole และโครงสร้างหนึ่งกด และการทดสอบการโปรแกรมในภาษา Go แบบชุดของโค้ด ส่งเสียมากถึง 1 ครั้ง

Workerman ได้เสนอการปรับปรุงประสิทธิภาพของแอปพลิเคชัน PHP ได้หลายเท่าหรือหลายสิบเท่า โค้ดที่ส่วนใหญ่ของ Workerman พัฒนาเพิ่มโครงสร้างหนึ่งกด อาจจะไม่ได้ทำให้ประสิทธิภาพดีขึ้น

หากระบบของคุณมีการเรียกข้อมูลที่ช้า เช่นการเรียก HTTP ภายนอก คุณอาจพิจารณาที่จะใช้โครงสร้างอย่างหนึ่งกดเพื่อปรับปรุงประสิทธิภาพ


