# الاسم

## الوصف:
```php
string Worker::$name
```

يُستخدم لتعيين اسم العامل (Worker) الحالي للتعرف على العملية عند تشغيل أمر الحالة (status). إذا لم يتم تعيينه، سيكون الاسم الافتراضي "none".

## مثال

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// تعيين اسم العامل (Worker) للمثال
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "بدء تشغيل العامل (Worker)...\n";
};
// تشغيل العامل (Worker)
Worker::runAll();
```
