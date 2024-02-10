# الحد الأقصى لحجم الحزمة

## الوصف:
```php
static int Connection::$defaultMaxPackageSize
```

هذا الخاصية هي خاصية ثابتة عالمية، تستخدم لتعيين أقصى حجم للحزمة التي يمكن لكل اتصال استقبالها. إذا لم يتم التعيين، فإن القيمة الافتراضية هي 10 ميجابايت.

إذا تم تحليل حزمة البيانات الواردة (قيمة إرجاع طريقة الإدخال في فئة البروتوكول) وتبين أن حجم الحزمة أكبر من ```Connection::$defaultMaxPackageSize```، سيتم اعتبارها بيانات غير قانونية وسيتم فصل الاتصال.

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// تعيين أقصى حجم للحزمة المستقبلة لكل اتصال إلى 1024000 بايت
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// تشغيل الخادم
Worker::runAll();
```
