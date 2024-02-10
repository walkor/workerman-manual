# getRemoteIp
## الوصف:
```php
string Connection::getRemoteIp()
```

الحصول على عنوان IP الخاص بالعميل لهذه الاتصال

## المعاملات

لا توجد معاملات


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "اتصال جديد من عنوان IP " . $connection->getRemoteIp() . "\n";
};
// تشغيل الخادم
Worker::runAll();
```
