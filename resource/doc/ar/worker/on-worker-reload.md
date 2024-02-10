# onWorkerReload
متطلبات ```（workerman >= 3.2.5）```
## الوصف:
```php
callback Worker::$onWorkerReload
```
هذه الميزة لا تستخدم بشكل شائع.

تعيين استدعاء يتم تنفيذه بواسطة Worker عند استلام إشارة reload.

يمكن استخدام استدعاء onWorkerReload للقيام بأشياء كثيرة، مثل إعادة تحميل ملفات تكوين الأعمال في حالة عدم الحاجة إلى إعادة تشغيل العملية.

**ملاحظة:**

سيتم الخروج وإعادة التشغيل افتراضيًا إذا تلقت العملية الفرعية إشارة reload، ليعيد العملية الجديدة تحميل الرمز الأساسي للأعمال لإكمال تحديث الرمز. لذلك، فإن خروج العملية الفرعية عقب استدعاء onWorkerReload بشكل فوري يعتبر أمراً طبيعيًا.

إذا كنت ترغب في أن تقوم العملية الفرعية بتنفيذ onWorkerReload فقط بعد استلام إشارة reload، دون الخروج، يمكنك ضبط خاصية reloadable لمثيل العملية الفرعية المناسب عند تهيئة مثيل Worker.

## معلمات الدالة الاستدعاءية

 ``` $worker ```

أي كائن Worker

## مثال

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// ضبط reloadable لتكون خاصية لا تُعاد تشغيل العملية الفرعية عند استلام إشارة reload
$worker->reloadable = false;
// بعد إعادة التحميل، يتم إخطار جميع عملاء الخادم بأن الخادم تم إعادة تحميله
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// تشغيل العميل
Worker::runAll();
```

ملحوظة: بالإضافة إلى استخدام الدالة المجهولة كاستدعاء، يمكنك أيضًا [الرجوع إلى هنا](../faq/callback_methods.md) لاستخدام أساليب استدعاء أخرى.
