يمكن استخدام المؤقت لإرسال البيانات بانتظام.

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');
// بعد بدء التشغيل ، يتم إرسال البيانات بانتظام إلى العملاء
$worker->onWorkerStart = function($worker){
    Timer::add(1, function()use($worker){
        foreach($worker->connections as $connection) {
            $connection->send('hello');
        }
    });
};
Worker::runAll();
```

توجد طريقة أخرى لإرسال البيانات إلى Workerman عند حدوث حدث في مشروع آخر. انظر [الأسئلة الشائعة - الإرسال في مشروع آخر](push-in-other-project.md)
