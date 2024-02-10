# إلغاء الاشتراك
**```(مطلوب Workerman الإصدار >= 3.3.0)```**

```php
void \Channel\Client::unsubscribe(string $event_name)
```
إلغاء الاشتراك في حدث معين، عند حدوث هذا الحدث، سيتم تجاهل استدعاء الرد الذي تم تسجيله باستخدام ```on($event_name, $callback)``` ```$callback```

### المعاملات
``` $event_name ```

اسم الحدث

### القيمة المُرجَعة
فارغ (void)


### مثال
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
    $event_name = 'user_login';
    Channel\Client::on($event_name, function($event_data){
        var_dump($event_data);
    });
    Channel\Client::unsubscribe($event_name);
};

Worker::runAll();
```
