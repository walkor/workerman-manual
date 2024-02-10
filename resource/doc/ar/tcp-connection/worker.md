# العامل
## الوصف:
```php
Worker Connection::$worker
```

هذا الخاصية هي خاصية للقراءة فقط، وهي العامل الحالي التي ينتمي إليها كائن الاتصال الحالي.


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// عندما يصل بيانات من عميل ما، سيتم إعادة توجيهها إلى جميع العملاء الآخرين المحتفظين بالعملية الحالية
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// تشغيل العامل
Worker::runAll();
```
