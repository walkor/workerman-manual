# الإغلاق
## الوصف:
```php
void Connection::destroy()
```

يُغلق الاتصال على الفور.

الاختلاف بينه وبين close هو أنه بعد استدعاء destroy، حتى إذا كان هناك بيانات في الذاكرة المؤقتة للإرسال للجهة الأخرى، فإن الاتصال سيتم إغلاقه على الفور وسيُشغَّل فورًا عنصر الاتصال ```onClose```.

## المعاملات

بدون معاملات

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // إذا حدث خطأ ما
    $connection->destroy();
};
// تشغيل الخادم
Worker::runAll();
```
