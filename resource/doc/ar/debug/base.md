# التصحيح الأساسي

يوجد لدى WorkerMan نوعان من أوضاع التشغيل، وضع التصحيح ووضع الخلفية (daemon).

عند تشغيل `php start.php start`، سيتم دخول وضع التصحيح. في هذا الوضع، ستكون عمليات الطباعة مثل `echo و var_dump و var_export` مرئية في نافذة الطرفية. يجب ملاحظة أنه عند تشغيل WorkerMan بهذه الطريقة، ستتوقف جميع العمليات عند إغلاق نافذة الطرفية.

أما عند تشغيل `php start.php start -d`، سيتم دخول وضع الخلفية، وهو الوضع الرسمي للتشغيل في الإنتاج، حيث لن يتأثر بإغلاق نافذة الطرفية.

إذا كنت ترغب في رؤية عمليات الطباعة مثل `echo و var_dump و var_export` في وضع الخلفية، يمكنك ضبط خاصية `Worker::$stdoutFile`. على سبيل المثال:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// توجيه الإخراج على الشاشة إلى الملف المحدد في Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

بهذه الطريقة، سيتم كتابة جميع عمليات الطباعة مثل `echo و var_dump و var_export` إلى الملف المحدد في `Worker::$stdoutFile`. يجب ملاحظة أن المسار المحدد في `Worker::$stdoutFile` يجب أن يكون له الصلاحيات الكافية للكتابة.
