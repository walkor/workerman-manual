# شرح

قام Workerman بتعزيز دعم خدمة HTTP ابتداءً من الإصدار 4.x. وقد قدم فئة الطلبات، وفئة الاستجابة، وفئة الجلسة إضافة إلى [SSE](SSE.md). إذا كنت ترغب في استخدام خدمة HTTP من Workerman، فإننا نوصي بشدة باستخدام Workerman 4.x أو أحدث.

**يرجى ملاحظة أن كل الأمثلة تعتمد على استخدام Workerman 4.x وغير متوافقة مع Workerman 3.x.**


# الحصول على كائن الجلسة
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// تشغيل الخادم
Worker::runAll();
```
**نصائح**
- يجب أن يتم التعامل مع الجلسة قبل استدعاء `$connection->send()`.
- ستُحفظ الجلسة تلقائيًا عند تدمير الكائن، لذا لا تقم بحفظ الكائن الذي يتم إرجاعه من `$request->session()` في مصفوفة عامة أو كعنصر فرعي في الفئة حتى لا يتم فقدان الجلسة.
- الجلسة تخزن افتراضياً في ملفات القرص، وإذا كنت ترغب في أداء أفضل يُوصى باستخدام Redis.


## الحصول على جميع بيانات الجلسة
```php
$session = $request->session();
$all = $session->all();
```
يُرجع مصفوفة. إذا لم يكن هناك أية بيانات للجلسة، سيتم إرجاع مصفوفة فارغة.

**مثال**
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

// تشغيل الخادم
Worker::runAll();
```


## الحصول على قيمة محددة من الجلسة
```php
$session = $request->session();
$name = $session->get('name');
```
سيتم إرجاع قيمة `null` إذا لم تكن البيانات موجودة.

يمكنك أيضًا تمرير قيمة افتراضية كمعامل ثانوي إلى الدالة `get`، وسيتم إرجاع القيمة الافتراضية إذا لم تكن القيمة موجودة في مصفوفة الجلسة. على سبيل المثال:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// تشغيل الخادم
Worker::runAll();
```


## تخزين الجلسة
استخدام الدالة `set` لتخزين البيانات.
```php
$session = $request->session();
$session->set('name', 'tom');
```
ليس لدى الدالة `set` أي قيمة إرجاع، ستقوم الجلسة بالحفظ تلقائيًا عند تدمير الكائن.

عند تخزين قيم متعددة، استخدم الدالة `put`.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
بنفس الطريقة، ليس لدى الدالة `put` أي قيمة إرجاع.

**مثال**
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

// تشغيل الخادم
Worker::runAll();
```


## حذف بيانات الجلسة
قم بحذف بيانات الجلسة الفردية باستخدام الدالة `forget`.
```php
$session = $request->session();
// حذف العنصر الفردي
$session->forget('name');
// حذف عناصر متعددة
$session->forget(['name', 'age']);
```

بالإضافة إلى ذلك، يمكنك استخدام الدالة `delete` التي تختلف عن الدالة `forget` حيث أن الأولى تحذف عنصرًا فرديًا فقط.
```php
$session = $request->session();
// مماثلة لـ $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// تشغيل الخادم
Worker::runAll();
```


## الحصول وحذف قيمة محددة من الجلسة
```php
$session = $request->session();
$name = $session->pull('name');
```
تعمل هذه الدالة مثل الشيفرة التالية
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
سيتم إرجاع قيمة `null` إذا كانت الجلسة غير موجودة.

**مثال**
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

// تشغيل الخادم
Worker::runAll();
```


## حذف جميع بيانات الجلسة
```php
$request->session()->flush();
```
لا توجد قيمة إرجاع، سيتم حذف بيانات الجلسة تلقائيًا عند تدمير الكائن.

**مثال**
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

// تشغيل الخادم
Worker::runAll();
```


## التحقق مما إذا كانت بيانات الجلسة موجودة
```php
$session = $request->session();
$has = $session->has('name');
```
سيتم إرجاع `false` إذا لم تكن بيانات الجلسة موجودة أو كانت القيمة مساوية لـ `null`.
```php
$session = $request->session();
$has = $session->exists('name');
```
تستخدم الشيفرة أعلاه للتحقق مما إذا كانت بيانات الجلسة موجودة، والفارق هو أنها ستُرجع `true` إذا كانت القيمة المقابلة في بيانات الجلسة تساوي `null`.
