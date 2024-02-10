# النوم
```php
int \Workerman\Timer::sleep(float $delay)
```
مماثل لوظيفة `sleep()` المدمجة في PHP، ولكن الفارق هو أن `Timer::sleep()` غير مانع (لا يحجب العملية الحالية).

> **ملاحظة**
> تتطلب هذه الميزة workerman>=5.0
> تتطلب هذه الميزة تثبيت composer require revolt/event-loop ^1.0.0، أو استخدام Swoole/Swow كدافع حدثي

### البارامترات
``` delay ```

الوقت الذي سيتم تنفيذ الوظيفة فيه، بوحدة الثواني، مع دعم الأرقام العشرية، يمكن تحديده بدقة إلى 0.001، أي بدقة تصل إلى مستوى الثانية.

### القيم المُعادة
لا توجد قيمة مُعادة

### مثال

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // تأخير إرسال البيانات لمدة 1.5 ثانية
    Timer::sleep(1.5);
    // إرسال البيانات
    $connection->send('hello workerman');
};

Worker::runAll();
```
