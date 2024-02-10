# onWorkerStart
## الوصف:
```php
callback Worker::$onWorkerStart
```

تعيين دالة الاستدعاء عند بدء تشغيل العمل الفرعي للعامل، حيث يتم تنفيذها عند بدء تشغيل كل عمل فرعي.

ملاحظة: يتم تشغيل onWorkerStart عند بدء تشغيل العمل الفرعي، وإذا تم تشغيل عدة عمليات فرعية (`$worker->count > 1`) ، فسيتم تشغيل كل عمل فرعي مرة واحدة، لذا سيتم تشغيلها ` $worker->count` مرات.
## معلومات الاستدعاء الراجعة

``` $worker ```

أي كائن Worker


## مثال

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "بدء تشغيل العامل...\n";
};
// تشغيل العامل
Worker::runAll();
```

ملاحظة: بالإضافة إلى استخدام الدالة المجهولة كدالة استدعاء، يمكنك أيضًا [الرجوع إلى هنا](../faq/callback_methods.md) لاستخدام طرق استدعاء أخرى.
