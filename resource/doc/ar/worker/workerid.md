# الهوية
يتطلب `workerman> = 3.2.1`

## الوصف:
```php
int Worker::$id
```

رقم تعريف معالج العمل الحالي، مع نطاق من `0` إلى `$worker->count-1`.

هذا الخاصية مفيدة جدًا لتمييز عمليات المعالج، على سبيل المثال إذا كان لديك مثيل واحد لمعالج يحتوي على عدة عمليات، وتريد فقط تعيين مؤقت في إحدى عملياته، بإمكانك التعرف على رقم تعريف العملية لتحقيق ذلك، مثل تعيين المؤقت فقط في العملية ذات الرقم الهوية 0 (انظر المثال).


## ملاحظة:
بعد إعادة تشغيل العملية، قيمة رقم تعريف العملية لا تتغير.

توزيع أرقام التعريف على العمليات مبني على كل مثيل من معالجات العمل. كل مثيل من معالجات العمل يبدأ من 0 بتعيين رقم تعريف لعملياته، لذلك قد يتكرر رقم التعريف بين مثيلات معالجات العمل، ولكنه لن يتكرر داخل مثيل واحد من معالجات العمل. كما في المثال التالي:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// المثيل 1 لديه 4 عمليات، وبالتالي أرقام تعريف العمليات 0, 1, 2, 3
$worker1 = new Worker('tcp://0.0.0.0:8585');
// تعيين تشغيل 4 عمليات
$worker1->count = 4;
// بعد تشغيل كل عملية، قم بطباعة رقم تعريف العملية الحالية، أي $worker1->id
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// المثيل 2 لديها 2 عمليات، وبالتالي أرقام تعريف العمليات 0, 1
$worker2 = new Worker('tcp://0.0.0.0:8686');
// تعيين تشغيل 2 عمليات
$worker2->count = 2;
// بعد تشغيل كل عملية، قم بطباعة رقم تعريف العملية الحالية، أي $worker2->id
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// تشغيل المعالجات
Worker::runAll();
```
الإخراج مشابه لـ
```sh
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

ملحوظة: نظام ويندوز لا يدعم تعيين عدد العمليات count، ولذلك رقم التعريف يكون فقط صفر. لا يمكن تشغيل نفس الملف لاستماع إلى عمليتين Worker في نظام ويندوز، لذلك لا يمكن تشغيل هذا المثال في نظام ويندوز.

## الأمثلة
مثال: مثيل واحد لديه 4 عمليات، فقط تعيين مؤقت في العملية ذات الرقم الهوية 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // تعيين المؤقت فقط في العملية ذات الرقم الهوية 0، ولا تقم بتعيين المؤقت في العمليات 1، 2، 3
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 عمليات معالج، تم تعيين المؤقت فقط في العملية ذات الرقم الهوية 0\n";
        });
    }
};
// تشغيل المعالجات
Worker::runAll();
```
