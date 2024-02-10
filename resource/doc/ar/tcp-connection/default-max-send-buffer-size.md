# defaultMaxSendBufferSize
## الوصف:
```php
static int Connection::$defaultMaxSendBufferSize
```

هذا الخاصية هي خاصية عامة ثابتة تُستخدم لتعيين حجم الذاكرة المؤقتة للإرسال على مستوى التطبيق لجميع الاتصالات. إذا لم يتم تعيين قيمة افتراضية، فإن القيمة الافتراضية هي `1 ميغابايت`. يمكن تعيين `Connection::$defaultMaxSendBufferSize` ديناميكياً، وسوف يكون مُطبقاً فقط على الاتصالات الجديدة التي تم إنشاؤها بعد التعيين.

هذه الخاصية تؤثر على [onBufferFull](../worker/on-buffer-full.md) callback.

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// تعيين حجم الذاكرة المؤقتة للإرسال على مستوى التطبيق لجميع الاتصالات
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // تعيين حجم الذاكرة المؤقتة للإرسال على مستوى التطبيق للاتصال الحالي، سيتم الكتابة فوق القيمة الافتراضية
    $connection->maxSendBufferSize = 4*1024*1024;
};
// تشغيل العامل
Worker::runAll();
```
