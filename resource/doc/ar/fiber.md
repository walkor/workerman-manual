يدعم workerman الفايبر (Fiber) ابتداءً من الإصدار 5.0.0 [Fiber](https://www.php.net/manual/zh/language.fibers.php)

> **ملاحظة**
> فايبر (Fiber) يتطلب PHP>=8.1 وتثبيت `composer require revolt/event-loop ^1.0.0`

### مقدمة

فايبر (Fiber) هو جلز حبكت اضم، يمكنها انقياد كود PHP ثم استئناف تنفيذه عند الحاجة. وظيفته الرئيسية هي تمكين المطورين من كتابة الكود بشكل متزامن للأكواد اللازمة للتنفيذ اللازم للفرع الشكلي الغير مانع، وهذا يعزز بشكل كبير صيانة الكود.

### مثال
لنقم بمقارنة استخدام الجلز (Fiber) والاستدعاء الغير متزامن.

**استخدام الاستدعاء الغير متزامن**
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
    // استدعاء واجهة HTTP
    $http->get('http://example.com/', function ($response) use ($connection) {
        // بعد فترة ثانية إرسال البيانات
        Timer::add(1, function() use ($connection, $response) {
            // إرسال البيانات إلى المتصفح
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**استخدام الجلز (Fiber)**
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
    // استدعاء واجهة HTTP
    $response = $http->get('http://example.com/');
    // تأخير لمدة ثانية
    Timer::sleep(1);
    // إرسال البيانات
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **ملاحظة**
> يجب تثبيت composer require workerman/http-client ^2.0.0

كلا الطريقتين تنفذان بشكل لازمي وغير مانع، ولكن استخدام الجلز يجعل الكود أسهل قراءة وصيانة بالمقارنة مع الاستدعاء الغير متزامن.

### ملاحظات حول الفايبر (Fiber)
* الفايبر (Fiber) لا يدعم جعل Pdo أو Redis أو وظائف الحجب الداخلية في PHP عمليات غير مانعة، وبمعنى آخر، استخدام هذه الامتدادات والوظائف مازال معتمد على الحجب.
* يوجد حالياً عميل فايبر (Fiber) يمكن استخدامه، منها [workerman/http-client](../components/workerman-http-client.md)، [workerman/redis](../components/workerman-redis.md)

# الجلز (Fiber) من خلال Swoole
يدعم workerman v5 استخدام Swoole كبرنامج جلز (Fiber) في التشغيل

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// يجب هنا تعيين Swoole يدوياً كبرنامج لتشغيل الجلز (Fiber)
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
**نصيحة**
* يُنصح باستخدام Swoole 5.0 أو أحدث
* جعل Swoole كبرنامج تشغيل الجلز (Fiber) يمكن workerman من دعم جلز Swoole
* استخدام Swoole كبرنامج تشغيل الجلز يسمح بعمل بدون تثبيت امتداد الحدث
* Swoole لا يحتوي افتراضياً على جلز أحادي، وبمعنى آخر، قائم على Pdo وRedis وقراءة/كتابة الملفات الداخلية في PHP بأنها استدعاء غير متزامن
* لتفعيل جلز أحادي، يجب استدعاء `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` يدوياً

لمزيد من المعلومات، يرجى الرجوع إلى [دليل Swoole](https://wiki.swoole.com/)

لمزيد من المعلومات، يرجى الرجوع إلى [الحدث المحرك](appendices/event.md)

# حول الجلز (Fiber)
أولاً، ليس من الضروري التمسك بشكل زائد بالجلز. عندما تكون قواعد البيانات، وRedis وغيرها من وحدات التخزين في الشبكة الداخلية، غالبًا ما يكون استدعاء العمليات المانعة أعلى أداءً من الجلز. من خلال [بيانات الضغط من techempower.com لمدة 3 سنوات](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db)، يظهر أداء استدعاء قاعدة بيانات workerman المانعة أفضل من استخدام مجموعة اتصالات قاعدة بيانات + الجلز في swoole، وحتى أداء إطار عمل جو بنسبة تقريبية مرتين.

لقد قامت workerman بتحسين أداء التطبيقات PHP بعدة أضعاف، وفي معظم مشاريع workerman، قد لا يكون استخدام الجلز له تأثير كبير في تحسين الأداء.
إذا كانت لديك استدعاءات بطيئة مثل استدعاءات HTTP الخارجية، يمكن التفكير في استخدام الجلز لتحسين الأداء.
