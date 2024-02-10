# إيقاف الكل
```php
void Worker::stopAll(void)
```

قم بإيقاف العملية الحالية والخروج. 

> **ملاحظة**
> `Worker::stopAll()` يستخدم لإيقاف العملية الحالية، بعد خروج العملية الرئيسية ستقوم بإطلاق عملية جديدة على الفور. إذا كنت ترغب في إيقاف خدمة webman بأكملها، يُرجى استدعاء `posix_kill(posix_getppid(), SIGINT)`

### المعاملات
بدون معاملات

### العودة
لا يوجد عودة

## مثال max_request

في المثال أدناه، يتم إيقاف عملية الفرع بعد معالجة 1000 طلب، من أجل إعادة تشغيل العملية بشكل جديد. يُستخدم هذا بشكل رئيسي لحل مشكلة تسرب الذاكرة الناشئة بسبب أخطاء رمز الأعمال في PHP مماثل لخاصية `max_request` في php-fpm.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// كل عملية تنفيذ تتعامل بحد أقصى مع 1000 طلب
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // عدد الطلبات التي تم معالجتها
    static $request_count = 0;

    $connection->send('hello http');
    // إذا وصل عدد الطلبات إلى 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * إيقاف عملية الفرع الحالية، وسيقوم العملية الرئيسية بإعادة تشغيل عملية جديدة على الفور
         * لذا ستتم إعادة تشغيل العملية
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
