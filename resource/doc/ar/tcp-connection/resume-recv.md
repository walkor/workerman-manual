استئنف الاستقبال

## الوصف:
```php
void Connection::resumeRecv(void)
```

يجعل الاتصال الحالي يستمر في استقبال البيانات. هذا الأسلوب مفيد جدًا لتحكم في حركة المرور للتحميل عند استخدامه بالتعاون مع Connection::pauseRecv.

## البارامترات

لا يوجد معلمات

## مثال

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // إضافة خاصية ديناميكية إلى كائن الاتصال لحفظ عدد الطلبات الواردة حاليًا
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // بمجرد استقبال 100 طلب، سيتم التوقف عن استقبال المزيد من البيانات
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // استئناف استقبال البيانات بعد 30 ثانية
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// تشغيل الخادم
Worker::runAll();
```

## انظر أيضًا
void Connection::pauseRecv(void) يجعل كائن الاتصال المقابل يتوقف عن استقبال البيانات
