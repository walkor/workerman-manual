# المستخدم

## الوصف:
```php
string Worker::$user
```

يحدد لمستخدم تعمل فيه شاشة العمل Worker الحالية. يمكن أن يتم تنفيذ هذا الخصائص فقط إذا كان المستخدم الحالي هو المدير النظام (root). إذا لم يتم تعيينها، سيتم تشغيله بشكل افتراضي بواسطة المستخدم الحالي. 

من الأفضل تعيين ```$user``` بمستخدم ذو صلاحيات منخفضة مثل www-data أو apache أو nobody.

ملاحظة: يجب تعيين هذه الخاصية قبل تشغيل ```Worker::runAll();``` لتكون سارية المفعول. ولا تدعم أنظمة ويندوز هذه الميزة.

## مثال

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// تعيين مستخدم التشغيل للشاشة الحالية
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "بدء تشغيل الشاشة...\n";
};
// تشغيل الشاشة
Worker::runAll();
```
