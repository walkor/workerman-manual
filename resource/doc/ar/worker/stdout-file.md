stdoutFile
## توضيح:
```php
static string Worker::$stdoutFile
```
هذا الخاصية هو خاصية ثابتة عامة. إذا تم تشغيله بطريقة البرنامج النصي (باستخدام```-d```), فإن جميع الانتاجات التي تم التوجيه إلى الطرفية (مثل echo var_dump وما إلى ذلك) سوف تُعيد توجيهها إلى ملف يتم تحديده بـstdoutFile.

إذا لم يتم تعيينه، وتم تشغيله بطريقة البرنامج النصي، فجميع الانتاجات من الطرفية تم تحويلها إلى `/dev/null` (أي تجاهل جميع الإخراج).

> ملاحظة: `/dev/null` هو ملف خاص في نظام Linux، حيث يعتبر بالفعل فوهة سوداء وجميع البيانات المُدخلة في هذا الملف ستُهمش. إذا كنت لا ترغب في تجاهل الإخراج، يمكنك تعيين `Worker::$stdoutFile` إلى مسار ملف عادي.

> ملاحظة: يجب تعيين هذا الخاصية قبل تشغيل `Worker::runAll();` من أجل أن تكون فعالة. وهذه الميزة غير مدعومة في نظام ويندوز.

## مثال
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// جميع الإخراجات ستتم حفظها في الملف /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "بدء العمل\n";
};
// تشغيل العامل
Worker::runAll();
```
