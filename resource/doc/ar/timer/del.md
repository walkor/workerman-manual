# حذف
```php
boolean \ Workerman \ Timer :: del (int $ timer_id)
```
حذف المؤقت 

### المعاملات
```timer_id```

معرف المؤقت، أي نوع صحيح يتم إرجاعه من واجهة الإضافة

### العائد
boolean


### مثال
```php
استخدام Workerman\Worker;
استخدام Workerman\Timer;
التضمين __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// فتح عدد من العمليات لتشغيل المهمة الزمنية ، يجب مراعاة مشكلة التنافس في عملية متعددة
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // كل 2 ثانية تعمل مرة واحدة
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // بعد 20 ثانية ، قم بتشغيل مهمة مرة واحدة ، وقم بحذف مهمة التوقيت كل 2 ثانية
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// تشغيل العامل
Worker::runAll();
```

### مثال (حذف المؤقت في استدعاء المؤقت)
```php
استخدام Workerman\Worker;
استخدام Workerman\Timer;
استخدام Workerman\Connection\TcpConnection;
التضمين __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // يرجى ملاحظة أنه يجب استخدام معرف المؤقت الحالي في استدعاء الرجوع باستخدام الطريقة المرجعية (&)
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // بعد تشغيل العد التاسع ، قم بحذف المؤقت
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// تشغيل العامل
Worker::runAll();
```
