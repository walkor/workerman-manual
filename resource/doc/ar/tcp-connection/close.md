# الإغلاق
## الوصف:
```php
void Connection::close(mixed $data = '')
```

يُغلق الاتصال بطريقة آمنة.

عند استدعاء close ، سيتم انتظار إرسال بيانات المخزن المؤقت لحين الانتهاء ثم يتم إغلاق الاتصال، وسيتم تطلق استدعاء `onClose` للاتصال.

## الباراميترات

 ``` $data ```

هذا هو باراميتر اختياري، وهو البيانات المراد إرسالها (إذا كان هناك بروتوكول محدد، سيتم استدعاء طريقة التشفير الخاصة بالبروتوكول لتعبئة البيانات `data` تلقائياً)، عند اكتمال إرسال البيانات يتم إغلاق الاتصال، ثم يتم تطلق استدعاء `onClose`.

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// تشغيل الخادم
Worker::runAll();
```
