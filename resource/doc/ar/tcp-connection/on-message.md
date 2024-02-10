# onMessage
## وصف:
```php
callback Connection::$onMessage
```


يقوم بنفس وظيفة استدعاء [Worker::$onMessage](../worker/on-message.md)، الاختلاف أنه يكون ساري المفعول فقط بالنسبة للاتصال الحالي، وبمعنى آخر يمكن تعيين استدعاء onMessage لاتصال معين.


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// عند حدوث حدث اتصال العميل
$worker->onConnect = function(TcpConnection $connection)
{
    // تعيين استدعاء onMessage للاتصال
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('receive success');
    };
};
// تشغيل الworker
Worker::runAll();
```

الشفرة أعلاه تقوم بنفس الوظيفة كالشفرة التالية

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// تعيين استدعاء onMessage مباشرة لجميع الاتصالات
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// تشغيل الworker
Worker::runAll();
```
