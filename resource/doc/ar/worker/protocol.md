# البروتوكول

المتطلبات: (workerman >= 3.2.7)

## الوصف:

```php
string Worker::$protocol
```

يُعين فئة البروتوكول الحالية لمثيل Worker.

ملاحظة: يمكن تحديد فئة معالجة البروتوكول مباشرةً عند تهيئة Worker في معلمات الاستماع. على سبيل المثال
```php
$worker = new Worker('http://0.0.0.0:8686');
```



## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// تشغيل الـ worker
Worker::runAll();
```

الشيفرة السابقة معادلة للشيفرة التالية

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * سيتم فحص ما إذا كان لدى المستخدم فئة بروتوكول مخصصة \Protocols\Http
 * إذا لم يكن هناك، سيتم استخدام فئة بروتوكول workerman المدمجة Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// تشغيل الـ worker
Worker::runAll();
```
