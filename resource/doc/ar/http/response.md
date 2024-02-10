# ملاحظة

- إذا كانت الاستجابة ليست chunk أو SSE ، فلا يُسمح بإرسال الرد مرة أخرى في طلب واحد ، أي أنه لا يُسمح بالاتصال المتكرر بـ `$connection->send()` في طلب واحد.
- يجب على كل طلب استدعاء `$connection->send()` مرة واحدة على الأقل لإرسال الاستجابة ، وإلا فإن العميل سينتظر بلا نهاية.

## الرد السريع
عندما لا يكون هناك حاجة لتغيير رمز الحالة HTTP (الافتراضي 200) ، أو تخصيص الهيدرات، الكوكيز، يمكنك ببساطة إرسال سلسلة إلى العميل لإكمال الاستجابة.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // إرسال "this is body" مباشرة للعميل
    $connection->send("this is body");
};

// تشغيل العامل
Worker::runAll();
```

## تغيير رمز الحالة
عند الحاجة إلى تخصيص رمز الحالة أو الهيدرات أو الكوكيز ، يجب استخدام فئة الاستجابة `Workerman\Protocols\Http\Response`. على سبيل المثال ، في المثال التالي عند الوصول إلى المسار `"/ 404"` يتم إرجاع رمز الحالة 404 ، ومحتوى الجسم هو `<h1>عذرًا، الملف غير موجود</h1>`.

**مثال**
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
        $connection->send(new Response(404, [], '<h1>عذرًا، الملف غير موجود</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// تشغيل العامل
Worker::runAll();
```
عندما يتم بالفعل تهيئة فئة `Response` ، يمكن تغيير رمز الحالة باستخدام الطريقة التالية.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## إرسال الهيدر
بنفس الطريقة ، يجب استخدام فئة الاستجابة `Workerman\Protocols\Http\Response` لإرسال الهيدرات.

**مثال**
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
        'X-Header-One' => 'قيمة الهيدر'
    ], 'this is body');
    $connection->send($response);
};

// تشغيل العامل
Worker::runAll();
```
عندما يتم بالفعل تهيئة فئة `Response` ، يمكن إضافة أو تغيير الهيدرات باستخدام الطريقة التالية.
```php
$response = new Response(200);
// إضافة أو تغيير هيدر
$response->header('Content-Type', 'text/html');
// إضافة أو تغيير أكثر من هيدر
$response->withHeaders([
    'Content-Type' => 'application/json',
    'X-Header-One' => 'قيمة الهيدر'
]);
$connection->send($response);
```

## إعادة توجيه
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

## إرسال الكوكيز
بنفس الطريقة ، يجب استخدام فئة الاستجابة `Workerman\Protocols\Http\Response` لإرسال الكوكيز.

**مثال**
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

// تشغيل العامل
Worker::runAll();
```

## إرسال ملف
بنفس الطريقة ، يجب استخدام فئة الاستجابة `Workerman\Protocols\Http\Response` لإرسال الملفات.

يمكن إرسال الملفات باستخدام الطريقة التالية
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- workerman يدعم إرسال ملفات كبيرة جدًا
- بالنسبة للملفات الكبيرة (أكبر من 2 ميجا بايت) ، لن يقوم workerman بقراءة الملف بأكمله دفعة واحدة إلى الذاكرة ، بل سيقوم بقراءة القطع بوقت مناسب وإرسالها
- سيقوم workerman بتحسين سرعة قراءة وإرسال الملفات وفقًا لسرعة الاستقبال من العميل ، وذلك لضمان إرسال الملف بأسرع ما يمكن مع تقليل استخدام الذاكرة إلى الحد الأدنى
- إرسال البيانات غير المانع ، ولن يؤثر على معالجة الطلبات الأخرى
- سيتم إضافة رأس `Last-Modified` تلقائيًا عند إرسال الملف لكي تعتمده الخادم في الطلبات المستقبلية لتوفير أداء أفضل
- سيتم إرسال الملف برأس `Content-Type` مناسب للمتصفح تلقائيًا
- في حالة عدم وجود الملف ، سيتم تحويله تلقائيًا إلى استجابة 404

**مثال**
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
    // تحقق إذا كانت هناك رأس if-modified-since للتحقق مما إذا كان الملف تم تعديله
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        //  إذا لم يتم التعديل على الملف فسيرد على طلب 304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    //  إذا تم تعديل الملف أو لم يكن هناك رأس if-modified-since سيرسل الملف
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// تشغيل العامل
Worker::runAll();
```

## إرسال بيانات http chunk
- يجب إرسال استجابة تحمل رأس `Transfer-Encoding: chunked` أولاً إلى العميل
- استخدام فئة `Workerman\Protocols\Http\Chunk` لإرسال بيانات chunk اللاحقة
- يجب إرسال chunk فارغة في النهاية لإنهاء الاستجابة

**مثال**
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
    // أولاً يجب إرسال استجابة تحمل رأس `Transfer-Encoding: chunked`
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // استخدام فئة `Workerman\Protocols\Http\Chunk` لإرسال بيانات chunk اللاحقة
    $connection->send(new Chunk('المرحلة الأولى من البيانات'));
    $connection->send(new Chunk('المرحلة الثانية من البيانات'));
    $connection->send(new Chunk('المرحلة الثالثة من البيانات'));
   //  النهاية يجب إرسال chunk فارغة لإنهاء الاستجابة
    $connection->send(new Chunk(''));
};

// تشغيل العامل
Worker::runAll();
```
