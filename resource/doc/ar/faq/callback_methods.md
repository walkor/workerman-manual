# أنواع عدة لكتابة الردود الفعل في PHP

PHP تقدم وسائل سهلة لكتابة الردود الفعل من خلال الدوال المجهولة، ولكن بالإضافة إلى الردود الفعل بواسطة الدوال المجهولة، هناك أنواع أخرى لكتابة الردود الفعل في PHP. فيما يلي أمثلة على أنواع كتابة الردود الفعل في PHP.

## ١. ردود فعل الدالة المجهولة
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// رد فعل الدالة المجهولة
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // إرسال "hello world" إلى المتصفح
    $connection->send('hello world');
};

Worker::runAll();
```

## ٢. رد فعل الدالة العادية
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// رد فعل الدالة المجهولة
$http_worker->onMessage = 'on_message';

// الدالة العادية
function on_message(TcpConnection $connection, Request $request)
{
    // إرسال "hello world" إلى المتصفح
    $connection->send('hello world');
}

Worker::runAll();
```

## ٣. استخدام الدوال في الصف كردود فعل
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
نص البدء start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// تحميل الصف MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// إنشاء كائن
$my_object = new MyClass();

// استدعاء دوال الصف
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

ملحوظة:
هذه الهيكلة لا تسمح بتهيئة الموارد (اتصال MySQL ، اتصال Redis ، اتصال Memcache الخ) في الدالة البنائية، لأن ```$my_object = new MyClass();``` تشتغل في العملية الرئيسية. كمثال على تهيئة اتصال MySQL في العملية الرئيسية، فإن هذه الموارد ستورث من العمليات الفرعية، حيث يمكن لكل عملية فرعية الوصول إلى اتصال قاعدة البيانات هذا، لنفترض اتصال MySQL، سيتم استلام خطأ غير متوقع مثل "mysql gone away" خطأ. 

هذا الهيكلة يمكن استخدامها إذا كان هناك حاجة لتهيئة الموارد في دالة البناء، يمكن استخدام الهيكلة التالية.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // فرضاً أن فئة اتصال قاعدة البيانات هي MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
نص البدء start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// تهيئة الصف في onWorkerStart
$worker->onWorkerStart = function($worker) {
    // تحميل الصف MyClass
    require_once __DIR__.'/MyClass.php';
    
    // إنشاء كائن
    $my_object = new MyClass();

    // استدعاء دوال الصف
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

في هذه الهيكلة، تشتغل الدالة onWorkerStart داخل العمليات الفرعية، وبذلك يتم إنشاء اتصال MySQL لكل عملية فرعية، مما يحل المشكلة المتعلقة بالاتصال المشترك.

## ٤. استخدام الدوال الثابتة في الصف كردود فعل
MyClass.php الثابتة
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){}
    public static function onConnect(TcpConnection $connection){}
    public static function onMessage(TcpConnection $connection, $message) {}
    public static function onClose(TcpConnection $connection){}
    public static function onWorkerStop(Worker $worker){}
}
```
نص البدء start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// تحميل الصف MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// استدعاء الدوال الثابتة في الصف.
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

Worker::runAll();
```

ملاحظة: طبقاً لآلية تشغيل PHP، إذا لم يكن هناك استدعاء "new" فلن يتم استدعاء الدالة البنائية، كما أن استخدام $this غير مسموح داخل الدوال الثابتة.
