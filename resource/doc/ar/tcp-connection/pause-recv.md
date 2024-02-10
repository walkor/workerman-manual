# إيقاف_الاستقبال
## الوصف:
```php
void Connection::pauseRecv(void)
```

يوقف الاتصال الحالي عن استقبال البيانات. لن يتم تشغيل استدعاء onMessage لهذا الاتصال. هذا الأسلوب مفيد جدًا لمراقبة تدفق البيانات الصادرة.

## الباراميترات

لا توجد معلمات

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // إضافة خاصية إلى كائن الاتصال ديناميكيًا لحفظ عدد الطلبات الواردة الحالية
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // بعد استقبال 100 طلب، سيتوقف الاتصال عن استقبال المزيد من البيانات
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// تشغيل العامل
Worker::runAll();
```

## انظر أيضًا
void Connection::resumeRecv(void) لاستئناف استقبال البيانات لكائن الاتصال المقابل
