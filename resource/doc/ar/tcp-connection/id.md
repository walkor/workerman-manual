# الهوية

## الوصف:
```php
int Connection::$id
```

هوية الاتصال. هذا هو عدد صحيح يتزايد ذاتياً.

ملاحظة: workerman يعمل بعمليات متعددة، وكل عملية تحتفظ بـ id للاتصال المتزايد ذاتياً، لذا قد تتكرر هويات الاتصال بين عمليات متعددة. إذا كنت بحاجة إلى هويات اتصال غير متكررة، يمكنك إعادة تعيين connection->id حسب الحاجة، مثل إضافة بادئة worker->id.

## راجع
[خاصية الاتصالات للعامل (Worker)](../worker/connections.md)

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// تشغيل العامل (Worker)
Worker::runAll();
```
