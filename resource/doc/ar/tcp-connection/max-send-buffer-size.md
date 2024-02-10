# الحد الأقصى لحجم البيانات المُرسَلة
## الوصف:
```php
int Connection::$maxSendBufferSize
```

كل اتصال يحتوي على مساحة تخزين مؤقتة مستقلة في طبقة التطبيق، إذا كانت سرعة استلام العميل أبطأ من سرعة إرسال الخادم، فإن البيانات ستخزن مؤقتًا في مساحة تخزين الطبقة العليا بانتظار الإرسال.

يُستخدم هذا الخاصية لتعيين حجم مساحة تخزين البيانات المُرسَلة في طبقة التطبيق للاتصال الحالي. إذا لم يتم تعيين قيمة، فسيكون الحجم الافتراضي هو [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md)(1 ميجابايت).

تؤثر هذه الخاصية في [onBufferFull](../worker/on-buffer-full.md) استدعاء الرد.

## مثال:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // تعيين حجم مساحة تخزين البيانات المُرسَلة في طبقة التطبيق للاتصال الحالي إلى 102400 بايت
    $connection->maxSendBufferSize = 102400;
};
// تشغيل الوركر
Worker::runAll();
```
