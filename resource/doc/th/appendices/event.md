# การเขับร่วมของ workerman ที่รองรับในปัจจุบัน

| ชื่อ | การขยายที่จำเป็น | รองรับ coroutine |  ลำดับความสำคัญ | เวอร์ชัน workerman |
|-----|------|--|-----|
| Workerman\Events\Select | ไม่มี | ไม่รองรับ | สนับสนุนตามค่าเริ่มต้นของคอร์ | >=3.0 |
| Workerman\Events\Revolt | event (เลือกได้) | รองรับ | ควรติดตั้ง [revolt/event-loop](https://github.com/revoltphp/event-loop) | >=5.0 |
| Workerman\Events\Event | event | ไม่รองรับ | สนับสนุนตามค่าเริ่มต้นของคอร์ | >=3.0 |
| Workerman\Events\Swoole | [swoole](https://github.com/swoole/swoole-src) | รองรับ | ควรตั้งค่าด้วยตนเอง | >=4.0 |
| Workerman\Events\Swow | [swow](https://github.com/swow/swow) | รองรับ | ควรตั้งค่าด้วยตนเอง | >=5.0 |

* การขับเคลื่อนแต่ละในคอร์ จะมีคุณสมบัติที่แตกต่างกัน เช่น การใช้ `Revolt` จะทำให้ workerman รองรับ [Fiber coroutine (เส้นใย)](https://www.php.net/manual/zh/language.fibers.php) ที่ภายใน PHP, การใช้ `Swoole` จะทำให้ workerman รองรับ coroutine ของ Swoole
* การขับเคลื่อนแต่ละกันเป็นการแยกทำงานกัน ยกตัวอย่างเช่น การใช้ `Revolt` ขณะที่ใช้ Fiber coroutine ไม่สามารถใช้ coroutine ของ Swoole หรือ Swow ได้
* `Revolt` ต้องการการติดตั้ง `composer require revolt/event-loop ^1.0.0`, เมื่อติดตั้งแล้วคอร์ของ workerman จะถูกตั้งค่าเป็นการขับเคลื่อนเหล่านี้อัตโนมัติเป็นอันดับแรก
* `Swoole` และ `Swow` ต้องการการตั้งค่า `Worker::$eventLoopClass` เพื่อให้มันเป็นผลสำเร็จ (ดูในช่วงประโยคถัดไป)
* swoole ค่าเริ่มต้นไม่ได้ตั้งค่าการเปิดใช้งาน [การทำงานร่วมกันอย่างรวดเร็วโดยอัตโนมัติ](https://wiki.swoole.com/#/runtime?id=runtime) นั่นหมายความว่าการเรียกใช้ Pdo, Redis, การอ่าน/เขียนไฟล์ที่ภายใน PHP ยังคงทำให้เกิดการบล็อก
* หากต้องการใช้งานการทำงานร่วมกันอย่างรวดเร็วของ swoole จะต้องเรียกใช้ `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` ด้วยตนเอง

> **ข้อควรระวัง**
> ส่วนขยาย swow จะทำการเปลี่ยนเปลงฟังก์ชันที่ซ่อนอยู่ภายใน PHP โดยอัตโนมัติ ซึ่งจะส่งผลให้ workerman ไม่สามารถตอบสนองคำขอและสัญญาณได้เมื่อเรียกใช้ swow เป็นตัวขับเคลื่อนเบื้องล่าง ดังนั้นหากคุณไม่ได้ใช้ swow เป็นการขับเคลื่อนเบื้องล่างคุณจำเป็นต้องคอมเม้น swow ออกจากไฟล์ php.ini ด้วย

อ่านเพิ่มเติมเกี่ยวกับ[coroutine ของ workerman](../fiber.md)

# การตั้งค่าการขับเคลื่อนเหตรของ workerman

ข้างล่างนี้คือการตั้งค่าการขับเคลื่อนเหตรของ workerman ด้วยตนเอง

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// การตั้งค่าการขับเคลื่อนเหตรของเบื้องล่างด้วยตนเอง
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
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
