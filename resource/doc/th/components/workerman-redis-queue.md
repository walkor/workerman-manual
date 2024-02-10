# workerman/redis-queue

คิวข้อความที่ใช้ Redis เพื่อรองรับการทำงานล่าช้าของข้อความ

## ที่อยู่ของโครเจ็ค:
https://github.com/walkor/redis-queue

## การติดตั้ง:
```composer require workerman/redis-queue```

## ตัวอย่าง
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // สมัครสมาชิก
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // สมัครสมาชิก
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // ตั้งเวลาส่งข้อความไปยังคิว
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## อินเทอร์เฟซ
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

สร้างอินสแตนซ์

  * `$address`  เช่น `redis://ip:6379` ต้องขึ้นต้นด้วย redis

  * `$options`  รวมถึงตัวเลือกต่อไปนี้:
    * `auth`: ข้อมูลการรับรองความถูกต้อง, เริ่มต้นคือ ''
    * `db`: db, เริ่มต้นคือ 0
    * `max_attempts`: จำนวนการลองลิงค์ภายหลังเมื่อคิวไม่สามารถทำงานได้, เริ่มต้นคือ 5
    * `retry_seconds`: ช่วงเวลาระหว่างการลองลิงค์, หน่วยเป็นวินาที, เริ่มต้นคือ 5

> การส่งล้มเหลวหมายถึงการธุรกรรมมีข้อยกเว้น `Exception` หรือ `Error` การส่งล้มเหลวจะนำข้อความไปยังคิวล่าช้าเพื่อลองลิงค์ซ้ำ จำนวนการลองลิงค์จะถูกควบคุมโดย `max_attempts` และระยะเวลาลองลิงค์จะถูกควบคุมโดย `retry_seconds` และ `max_attempts` ตัวอย่างเช่น `max_attempts` เป็น 5 `retry_seconds` เป็น 10 ช่วงเวลาที่ 1 ของการลองลิงค์ครั้งที่ 1 คือ `1*10` วินาที ช่วงเวลาที่ 2 ของการลองลิงค์ครั้งที่ 2 คือ `2*10` วินาที ช่วงเวลาที่ 3 ของการลองลิงค์ครั้งที่ 3 คือ `3*10` วินาที และต่อไป ถึงการลองลิงค์ 5 ครั้ง หากเกินการตั้งค่า `max_attempts` จะถูกนำข้อความไปยังคิวล้มเหลวที่มีชื่อ key เป็น `{redis-queue}-failed` (ก่อนเวอร์ชัน 1.0.5 เป็น `redis-queue-failed`)

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

ส่งข้อความไปยังคิวหนึ่งรายการ

* `$queue` ชื่อคิว, ประเภท `String`
* `$data` ข้อความที่จะเผยแพร่, สามารถเป็นอาร์เรย์หรือสตริง, ประเภท `Mixed`
* `$dely` เวลาล่าช้าก่อนที่จะตัดสินใจทำรายการ, หน่วยวินาที, เริ่มต้นคือ 0, ประเภท `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

สมัครสมาชิกกับคิวหรือคิวหลายรายการ

* `$queue` ชื่อคิว, สามารถเป็นสตริงหรืออาร์เรย์ที่มีชื่อคิวหลายรายการ
* `$callback` ฟังก์ชัน callback รูปแบบเป็น `function (Mixed $data)` ที่ `$data` เป็น `send($queue, $data)`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

ยกเลิกการสมัครสมาชิก

* `$queue` ชื่อคิวหรืออาร์เรย์ที่มีชื่อคิวหลายรายการ

-------------------------------------------------------

## การส่งข้อความไปยังคิวในสภาพแวดล้อมนอก workerman
บางครั้งมีโปรเจคที่ทำงานในสภาพแวดล้อมของ apache หรือ php-fpm และไม่สามารถใช้โครเจ็ค workerman/redis-queue ได้ สามารถอ้างอิงฟังก์ชันดังต่อไปนี้เพื่อทำการส่งได้
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; //ก่อนเวอร์ชัน 1.0.5 เป็น redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';//ก่อนเวอร์ชัน 1.0.5 เป็น redis-queue-delayed
    
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```
ที่นั้นพารามิเตอร์ `$redis` เป็นตัวอย่างของ redis instance เช่นการใช้งานของการขยาย redis คลาสเป็นดังนี้:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
```
