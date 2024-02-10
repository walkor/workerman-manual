# getRemotePort
## الوصف:
```php
int Connection::getRemotePort()
```

الحصول على منفذ العميل الذي يتصل بالاتصال

## المعاملات

لا تحتاج لمعاملات


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "اتصال جديد من عنوان " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// تشغيل العامل
Worker::runAll();
```
