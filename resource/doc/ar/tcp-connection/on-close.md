# onClose
## الوصف:
```php
callback Connection::$onClose
```

هذا التابع يعمل بنفس تأثير التابع [Worker::$onClose](../worker/on-close.md)، لكن الاختلاف هو أنه يكون ساري المفعول فقط على الاتصال الحالي، ويعني أنه يمكن تعيين تابع onClose لاتصال معين.

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// عند حدوث حدث الاتصال
$worker->onConnect = function(TcpConnection $connection)
{
    // تعيين تابع onClose للاتصال
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "تم إغلاق الاتصال\n";
    };
};
// تشغيل العمال
Worker::runAll();
```

الكود أعلاه له نفس النتيجة مع الكود التالي

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// تعيين تابع onClose لجميع الاتصالات
$worker->onClose = function(TcpConnection $connection)
{
    echo "تم إغلاق الاتصال\n";
};
// تشغيل العمال
Worker::runAll();
```
