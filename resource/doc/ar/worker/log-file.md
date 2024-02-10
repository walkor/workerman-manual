# ملف السجل
## الوصف:
```php
static string Worker::$logFile
```

يُستخدم لتحديد موقع ملف السجل الخاص ب Workerman.

يُسجل في هذا الملف أحداث Workerman ذات الصلة الخاصة به، بما في ذلك بدء التشغيل وإيقاف التشغيل وما إلى ذلك.

إذا لم يتم تعيين موقع ملف السجل، فإن اسم الملف الافتراضي سيكون workerman.log وسيتم وضعه في الدليل العلوي ل Workerman.

**ملاحظة:**

يُسجل هذا الملف الخاص فقط بأحداث Workerman ذات الصلة بالبدء والإيقاف وما إلى ذلك، ولا يتضمن أي سجلات خاصة بالأعمال.

يمكن لفصيل الأعمال استخدام أدواته الخاصة مثل [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) أو [error_log](https://php.net/manual/zh/function.error-log.php) لتنفيذ ذلك.

## المثال

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// تشغيل الخادم
Worker::runAll();
```
