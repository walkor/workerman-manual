# ووركرمان / رديس-كيو

قائمة رسائل قائمة على رديس، تدعم معالجة الرسائل المؤجلة.

## عنوان المشروع:
https://github.com/walkor/redis-queue

## التثبيت:
```composer require workerman/redis-queue```

## مثال
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // الاشتراك
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // الاشتراك
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // إرسال الرسائل إلى قائمة الرسائل بانتظام
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## واجهة برمجة التطبيقات
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

إنشاء المثيل

  * `$address`  مثل `redis://ip:6379`، يجب أن تبدأ بـ redis. 

  * `$options`  تتضمن الخيارات التالية:
    * `auth`: معلومات المصادقة، الافتراضي ''
    * `db`: db، الافتراضي 0
    * `max_attempts`: عدد مرات إعادة المحاولة بعد فشل الاستهلاك، الافتراضي 5
    * `retry_seconds`: فاصل زمني لإعادة المحاولة، بالثواني. الافتراضي 5

> فشل الاستهلاك يعني رمي ثائر الأعمال `Exception` أو `Error`. بعد الفشل، يتم وضع الرسالة في قائمة الانتظار للمحاولة مرة أخرى، عدد المرات التي يتم فيها إعادة المحاولة يتم التحكم بها من خلال `max_attempts`، ويتم التحكم في فاصل الزمن لإعادة المحاولة من خلال `retry_seconds` و `max_attempts` معًا. على سبيل المثال، إذا كان `max_attempts` هو 5، و `retry_seconds` هو 10، فإن الفاصل الزمني للمحاولة الأولى هو `1*10` ثانية، والفاصل الزمني للمحاولة الثانية هو `2*10` ثانية، والفاصل الزمني للمحاولة الثالثة هو `3*10` ثانية، وهكذا حتى يصل إلى 5 مرات. إذا تجاوز عدد مرات إعادة المحاولة القيمة المحددة في `max_attempts`، فإن الرسالة يتم وضعها في مفتاح يسمى `{redis-queue}-failed` (قبل الإصدار 1.0.5 كان اسمه `redis-queue-failed`) في قائمة الفشل

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

إرسال رسالة إلى قائمة

* `$queue` اسم القائمة، نوع `String`
* `$data` الرسالة المحددة للنشر، يمكن أن تكون مصفوفة أو سلسلة، نوع `Mixed`
* `$dely` وقت تأخير الاستهلاك، بالثواني، الافتراضي 0، نوع `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

الاشتراك في قائمة واحدة أو عدة قوائم

* `$queue` اسم القائمة، يمكن أن يكون سلسلة أو مصفوفة تحتوي على أسماء قوائم متعددة
* `$callback` الدالة المستدعاة، بتنسيق `function (Mixed $data)`، حيث `$data` هو `send($queue, $data)` في`$data`.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

إلغاء الاشتراك

* `$queue` اسم القائمة أو مصفوفة تحتوي على أسماء قوائم متعددة

-------------------------------------------------------

## في بيئة workerman أخرى لإرسال رسائل إلى القائمة
في بعض الأحيان يتم تشغيل بعض المشاريع في بيئة apache أو php-fpm، ولا يمكن استخدام مشروع workerman/redis-queue، يمكن الرجوع إلى الدالة التالية لتحقيق الإرسال
```php
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; // قبل الإصدار 1.0.5 كان اسمهredis-queue-waiting
    $queue_delay = '{redis-queue}-delayed';// قبل الإصدار 1.0.5 كان اسمهredis-queue-delayed
    
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
حيث، المعلمات `$redis` هي مثيل رديس. مثلا استخدام امثلة التوسيع رديس مثل:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
````
