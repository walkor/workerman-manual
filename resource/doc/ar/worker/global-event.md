# الحدث العالمي

## الوصف:
```php
static Event Worker::$globalEvent
```

هذا الخاصية هي خاصية ثابتة عالمية، وهي عبارة عن مثيل عالمي لحلقة الأحداث، يمكنك تسجيل أحداث قراءة وكتابة وحدث الإشارة لمعرف الملف أو حدث الإشارة.


## مثال

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // عندما يتلقى العملية إشارة SIGALRM ، اطبع بعض المعلومات
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "احصل على إشارة SIGALRM\n";
    });
};
// تشغيل العامل
Worker::runAll();
```

## اختبار
بمجرد تشغيل Workerman ، ستظهر تعليقات حول معرف العملية الحالي (رقم). قم بتشغيل الأمر في سطر الأوامر
``` 
kill -SIGALRM معرف العملية
``` 
ستظهر الخادم إخراجًا يقول
```
احصل على إشارة SIGALRM
```
