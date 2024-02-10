# شرح
بدءًا من الإصدار 4.x ، قام Workerman بتعزيز دعم خدمة HTTP. وقد قدم فئة الطلب والاستجابة وفئة الجلسة و[SSE](SSE.md). إذا كنت ترغب في استخدام خدمة HTTP في Workerman ، فإننا نوصي بشدة باستخدام Workerman 4.x أو أحدث.

**يرجى ملاحظة أن كل ما يلي هو لاستخدام Workerman 4.x ، ولا يتوافق مع Workerman 3.x.**

## تغيير محرك تخزين الجلسة
قدم Workerman محرك تخزين الملفات ومحرك تخزين redis للجلسة. يتم استخدام محرك تخزين الملفات افتراضيا. إذا كنت ترغب في التغيير إلى محرك تخزين redis ، يرجى الرجوع إلى الكود التالي.

```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// تكوين redis
$config = [
    'host'     => '127.0.0.1', // المعلمة المطلوبة
    'port'     => 6379,        // المعلمة المطلوبة
    'timeout'  => 2,           // المعلمة الاختيارية
    'auth'     => '******',    // المعلمة الاختيارية
    'database' => 1,           // المعلمة الاختيارية
    'prefix'   => 'session_'   // المعلمة الاختيارية
];
// استخدام Workerman\Protocols\Http\Session::handlerClass لتغيير فئة المحرك الداخلي للجلسة
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## تعيين موقع تخزين الجلسة
عند استخدام محرك التخزين الافتراضي ، يتم تخزين بيانات الجلسة افتراضياً في القرص الثابت ، والموقع الافتراضي هو الموقع الذي يتم إرجاعه بواسطة `session_save_path()`.
يمكنك استخدام الطريقة التالية لتغيير الموقع.

```php
use Workerman\Worker;
use \Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// تعيين موقع تخزين ملف الجلسة
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// تشغيل العامل
Worker::runAll();
```

## تنظيف ملفات الجلسة
عند استخدام محرك تخزين الجلسة الافتراضي على القرص الثابت ، ستكون هناك ملفات جلسة متعددة في القرص الثابت.
سوف يقوم Workerman بتنظيف الملفات الزائدة للجلسة وفقًا للإعدادات session.gc_probability, session.gc_divisor, session.gc_maxlifetime الموجودة في php.ini. لمزيد من المعلومات حول هذه الإعدادات ، يرجى الرجوع إلى [دليل PHP](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability)

## تغيير محرك التخزين
بالإضافة إلى محرك تخزين ملفات الجلسة ومحرك تخزين redis الافتراضيين ، يسمح لك Workerman بإضافة محرك تخزين جلسة جديد باستخدام واجهة [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php) القياسية ، مثل محرك تخزين جلسة MongoDB أو محرك تخزين جلسة MySQL.

**عملية إضافة محرك تخزين جلسة جديدة**
  1. تنفيذ واجهة [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php)
  2. استخدم الطريقة `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` لاستبدال واجهة SessionHandler الأساسية
 
**تنفيذ واجهة SessionHandlerInterface**

يجب تنفيذ محرك تخزين الجلسة المخصص وفقًا لواجهة SessionHandlerInterface. تشمل هذه الواجهة الأساليب التالية:
```php
SessionHandlerInterface {
    /* Methods */
    abstract public function read ( string $session_id ) : string
    abstract public function write ( string $session_id , string $session_data ) : bool
    abstract public function destroy ( string $session_id ) : bool
    abstract public function gc ( int $maxlifetime ) : int
    abstract public function close ( void ) : bool
    abstract public function open ( string $save_path , string $session_name ) : bool
}
```
**تفسير واجهة SessionHandlerInterface**
 - يُستخدم الطريقة read لقراءة جميع بيانات الجلسة المرتبطة بمعرف الجلسة من التخزين. يرجى عدم تنفيذ عمليات فك تسلسل للبيانات ، فالإطار سيقوم بذلك تلقائياً.
 - يُستخدم الطريقة write لكتابة بيانات الجلسة المرتبطة بمعرف الجلسة إلى التخزين. يرجى عدم تنفيذ عمليات تسلسل للبيانات ، فالإطار قد قام بذلك بالفعل.
 - يُستخدم الطريقة destroy لتدمير بيانات الجلسة المرتبطة بمعرف الجلسة.
 - يُستخدم الطريقة gc لحذف بيانات الجلسة المنتهية صلاحيتها ، يجب على التخزين حذف جميع بيانات الجلسة التي تم تعديلها آخر مرة قبل maxlifetime.
 - لا يتطلب الطريقة close أي عمليات ، يجب أن تقوم بإرجاع true مباشرة.
 - لا يتطلب الطريقة open أي عمليات ، يجب أن تقوم بإرجاع true مباشرة.

**استبدال واجهة السائق الأساسية**

بعد تنفيذ واجهة SessionHandlerInterface ، قم بتغيير سائق الجلسة الأساسي باستخدام الطريقة التالية.

```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name اسم فئة سائق الجلسة المطبق لواجهة SessionHandlerInterface. إذا كان هناك مساحة اسم ، فيجب عليك تضمينها بالكامل.
 - $config هو معاملات البناء لفئة SessionHandler.

**التنفيذ المحدد**

*يرجى ملاحظة أن فئة MySessionHandler صُممت فقط لتوضيح عملية تغيير سائق جلسة الأساسية ، لا يمكن استخدامها في البيئة الإنتاجية.*
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

//  نفترض أن فئة SessionHandler المنفذة تتطلب بعض المعلمات
$config = ['host' => 'localhost'];
// استخدام Workerman\Protocols\Http\Session::handlerClass($class_name, $config) لتغيير فئة السائق الداخلية للجلسة
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
