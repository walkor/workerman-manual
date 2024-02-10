# workerman/crontab

# الوصف
`workerman/crontab` هو برنامج مهام مجدولة يعتمد على workerman، مشابه لـ crontab في نظام لينكس. `workerman/crontab` يدعم الجدولة بدقة الثانية.

> لاستخدام `workerman/crontab`، يجب تعيين المنطقة الزمنية في PHP أولاً، وإلا قد لا تكون النتائج متوقعة.

## شرح الوقت
```plaintext
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ يوم الأسبوع (0 - 6) (الأحد=0)
|   |   |   |   +------ شهر (1 - 12)
|   |   |   +-------- يوم الشهر (1 - 31)
|   |   +---------- ساعة (0 - 23)
|   +------------ دقيقة (0 - 59)
+-------------- ثانية (0-59)[يمكن حذفها، إذا لم تكن موجودة فإن أقل وحدة زمنية هي الدقيقة]
```

# التثبيت
```plaintext
composer require workerman/crontab
```

# مثال
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// تعيين المنطقة الزمنية لتجنب عدم تطابق النتائج بالتوقعات
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // تنفيذ كل دقيقة في الثانية 1.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // تنفيذ كل 7:50 صباحًا يومياً، يجب ملاحظة أن هنا تم حذف وحدة الثانية.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> ملاحظة: لن يتم تنفيذ المهام المجدولة على الفور، بل ستبدأ تنفيذ كل المهام المجدولة عند دخول الدقيقة التالية.

# واجهة برمجة التطبيقات
**Crontab::destroy()**

تدمير الموقت
```plaintext
$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();
```
