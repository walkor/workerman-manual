# شرح
منذ الإصدار 4.x ، قام Workerman بتعزيز دعم خدمة HTTP. قدمت فئات الطلب والاستجابة والجلسة و[SSE](SSE.md). إذا كنت ترغب في استخدام خدمة HTTP في Workerman ، فإننا نوصي بشدة باستخدام إصدار Workerman 4.x أو الإصدارات الأحدث.

**يرجى ملاحظة أن كل ما يلي هو لاستخدام إصدار Workerman 4.x وغير متوافق مع إصدار Workerman 3.x.**

## الحصول على كائن الطلب
يجب الحصول على كائن الطلب في وظيفة رد الرسائل onMessage. تقوم الإطارات تلقائيًا بتمرير كائن الطلب كمعامل ثانوي إلى وظيفة الارجاع.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request هو كائن الطلب ، في هذا المكان لم يتم تنفيذ أي عمل على كائن الطلب مباشرةً يتم إرجاع hello مباشرة للمتصفح
    $connection->send("hello");
};

// تشغيل العامل
Worker::runAll();
```

عندما يقوم المتصفح بزيارة `http://127.0.0.1:8080` ستعود الإجابة `hello`.

## الحصول على معلمات الطلب get
**الحصول على مصفوفة get كاملة**
```php
$get = $request->get();
```
إذا لم تكن هناك معلمات get في الطلب ، فسيرجع مصفوفة فارغة.

**الحصول على قيمة محددة من مصفوفة get**
```php
$name = $request->get('name');
```
إذا كانت مصفوفة get لا تحتوي على هذه القيمة ، فسيرجع قيمة null.

يمكنك أيضًا تمرير قيمة افتراضية كمعامل ثانوي لأسلوب get. إذا لم تجد قيمة مطابقة في مصفوفة get ، فسيرجع القيمة الافتراضية. على سبيل المثال:
```php
$name = $request->get('name', 'tom');
```

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->get('name'));
};

// تشغيل العامل
Worker::runAll();
```

عندما يقوم المتصفح بزيارة `http://127.0.0.1:8080?name=jerry&age=12` ستعود الإجابة `jerry`.

## الحصول على معلمات الطلب post
**الحصول على مصفوفة post كاملة**
```php
$post = $request->post();
```
إذا لم يكن هناك معلمات post في الطلب ، فسيرجع مصفوفة فارغة.

**الحصول على قيمة محددة من مصفوفة post**
```php
$name = $request->post('name');
```
إذا كانت مصفوفة post لا تحتوي على هذه القيمة ، فسيرجع قيمة null.

مثل اسلوب get ، يمكنك أيضًا تمرير قيمة افتراضية كمعامل ثانوي لاسلوب post. إذا لم تجد قيمة مطابقة في مصفوفة post ، سيرجع القيمة الافتراضية. على سبيل المثال:
```php
$name = $request->post('name', 'tom');
```

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// تشغيل العامل
Worker::runAll();
```

## الحصول على جسم الطلب النصي الخام
```php
$post = $request->rawBody();
```
هذه الوظيفة مشابهة لعملية `file_get_contents("php://input")` في `php-fpm`. تُستخدم للحصول على جسم الطلب النصي الخام في HTTP. يتم استخدامها عند الحصول على بيانات طلب post بتنسيق غير `application/x-www-form-urlencoded`.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// تشغيل العامل
Worker::runAll();
```

## الحصول على الترويسة
**الحصول على مصفوفة الترويسة كاملة**
```php
$headers = $request->header();
```
إذا كانت الطلب لا تحتوي على معلمات الترويسة ، فسيرجع مصفوفة فارغة. يجب ملاحظة أن جميع المفاتيح بالحالة الصغيرة.

**الحصول على قيمة محددة من مصفوفة الترويسة**
```php
$host = $request->header('host'); 
```
إذا كانت مصفوفة الترويسة لا تحتوي على هذه القيمة فسيرجع قيمة null. يجب ملاحظة أن جميع المفاتيح بالحالة الصغيرة.

مثل اسلوب get، يمكنك أيضًا تمرير قيمة افتراضية كمعامل ثانوي لاسلوب header. إذا لم تجد قيمة مطابقة في مصفوفة الترويسة ، سيرجع القيمة الافتراضية. على سبيل المثال:
```php
$host = $request->header('host', 'localhost');
```

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// تشغيل العامل
Worker::runAll();
```

## الحصول على الكوكي
**الحصول على مصفوفة الكوكي بالكامل**
```php
$cookies = $request->cookie();
```
إذا لم تحتوي الطلب على معلمات الكوكي فسيرجع مصفوفة فارغة.

**الحصول على قيمة محددة من مصفوفة الكوكي**
```php
$name = $request->cookie('name');
```
إذا لم تحتوي مصفوفة الكوكي على هذه القيمة فسيرجع قيمة null.

مثل اسلوب get ، يمكنك أيضًا تمرير قيمة افتراضية كمعامل ثانوي لأسلوب الكوكي. إذا لم تجد قيمة مطابقة في مصفوفة الكوكي ، سيرجع القيمة الافتراضية. على سبيل المثال:
```php
$name = $request->cookie('name', 'tom');
```

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// تشغيل العامل
Worker::runAll();
```

## الحصول على ملفات الرفع
**الحصول على مصفوفة الرفع بالكامل**
```php
$files = $request->file();
```
سيعود تنسيق الملفات المشابه للنمط التالي:

```php
array(
    'avatar' => array(
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' => array(
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
حيث:

- name هو اسم الملف
- tmp_name هو موقع الملف المؤقت على القرص
- size هو حجم الملف
- error هو [كود الخطأ](https://www.php.net/manual/zh/features.file-upload.errors.php)
- type هو نوع ملف mime.

**ملاحظة:**

- حجم الملف المرتفع مقيد بـ [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md) ، الافتراضي 10M، يمكن تغييره.

- يتم تنظيف الملفات تلقائيًا بعد انتهاء الطلب.

- إذا كان الطلب لا يحتوي على ملفات رفع ، سيرجع مصفوفة فارغة.

### الحصول على ملف رفع معين
```php
$avatar_file = $request->file('avatar');
```
سيعود نمط مماثل للنمط التالي
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream'
  )
```
إذا لم يكن هناك ملف رفع، سيرجع قيمة null.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// تشغيل العامل
Worker::runAll();
```

## الحصول على المضيف
الحصول على معلومات المضيف في الطلب.
```php
$host = $request->host();
```
إذا كان عنوان الطلب ليس على المنفذ القياسي 80 أو 443، فقد يتم تضمين معلومات المنفذ في معلومات المضيف، على سبيل المثال: `example.com:8080`. إذا لم تكن بحاجة إلى معلومات المنفذ، يمكن تمرير القيمة true كمعامل أولي.

```php
$host = $request->host(true);
```
**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->host());
};

// تشغيل العامل
Worker::runAll();
```

عندما يقوم المتصفح بزيارة `http://127.0.0.1:8080?name=tom` ستعود الإجابة `127.0.0.1:8080`.
## الحصول على طريقة الطلب
```php
$method = $request->method();
```
قد تكون القيمة المُُرجَعة `GET` أو `POST` أو `PUT` أو `DELETE` أو `OPTIONS` أو `HEAD`.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// تشغيل الوركر
Worker::runAll();
```

## الحصول على URI الطلب
```php
$uri = $request->uri();
```
تُرجع URI للطلب بما في ذلك جزء الـ path وجزء queryString.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// تشغيل الوركر
Worker::runAll();
```
عند زيارة المتصفح لـ `http://127.0.0.1:8080/user/get.php?uid=10&type=2` سيتم إرجاع `/user/get.php?uid=10&type=2`.

## الحصول على مسار الطلب
```php
$path = $request->path();
```
تُرجع جزء الـ path للطلب.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// تشغيل الوركر
Worker::runAll();
```
عند زيارة المتصفح لـ `http://127.0.0.1:8080/user/get.php?uid=10&type=2` سيتم إرجاع `/user/get.php`.

## الحصول على سلسلة الاستعلام للطلب
```php
$query_string = $request->queryString();
```
تُرجع جزء الـ queryString للطلب.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// تشغيل الوركر
Worker::runAll();
```
عند زيارة المتصفح لـ `http://127.0.0.1:8080/user/get.php?uid=10&type=2` سيتم إرجاع `uid=10&type=2`.

## الحصول على إصدار بروتوكول الطلب
```php
$version = $request->protocolVersion();
```
تُرجع سلسلة نصية `1.1` أو `1.0`.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// تشغيل الوركر
Worker::runAll();
```

## الحصول على معرف الجلسة للطلب
```php
$sid = $request->sessionId();
```
تُرجع سلسلة نصية مكونة من الحروف والأرقام.

**مثال**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// تشغيل الوركر
Worker::runAll();
```
