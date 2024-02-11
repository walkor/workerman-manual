الخاصية pidFile

## الوصف:
```php
static string Worker::$pidFile
```

يُفضل عدم ضبط هذه الخاصية إلا إذا كان هناك حاجة خاصة لذلك.

هذه الخاصية هي خاصية ثابتة على مستوى العمل، تُستخدم لتحديد مسار ملف pid الخاص بعمليات Workerman.

إعداد هذه الخاصية يكون مفيدًا في التحكم، على سبيل المثال، يُمكن وضع ملف pid الخاص بـ Workerman في دليل ثابت، مما يُسهل على بعض برامج التحكم قراءة ملفات pid وبالتالي رصد حالة عمليات Workerman.

إذا لم يُحدد، سيقوم Workerman تلقائيًا بإنشاء ملف pid في مكان يتوازى مع مجلد Workerman (يرجى ملاحظة أن الإصدارات قبل Workerman 3.2.3 كانت تُنشئ ملفات pid بشكل افتراضي في ```sys_get_temp_dir()```)، ومن أجل تجنب تضارب ملفات pid الناتج عن تشغيل عدة نسخ من Workerman، يتم تضمين مسار Workerman الحالي في ملفات pid التي يُنشأها Workerman.

ملاحظة: يجب ضبط هذه الخاصية قبل تشغيل ```Worker::runAll();``` حتى تكون فعالة. هذه الخاصية غير مدعومة في نظام windows.

## مثال:

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "بدء العمليات";
};
// تشغيل العملية
Worker::runAll();
```
