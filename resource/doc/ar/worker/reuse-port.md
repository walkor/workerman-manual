# استخدام ReusePort
> **ملاحظة**
> يتطلب Workerman >= 3.2.1 و PHP >= 7.0 ، ويُعتبر النظامين Windows و Mac OS غير مدعومين لهذه الميزة

## الوصف:

```php
bool Worker::$reusePort
```

يُعيّن ما إذا كان العامل الحالي يُفعّل إعادة استخدام منفذ الاستماع (خيار SO_REUSEPORT لمأخذ الاتصال) أم لا.

بعد تمكين إعادة استخدام منفذ الاستماع، يُسمح لعدة عمليات غير ذات صلة بالعمل بالاستماع إلى نفس المنفذ ويقوم نظام النواة بتوزيع الأعباء لتقرير أي عملية ستتعامل مع اتصال المأخذ، مما يجنب تأثير الحالة الجائية، ويمكنه تحسين أداء التطبيقات ذات الاتصالات المتعددة القصيرة لعملية متعددة.

**ملاحظة:** هذه الميزة تتطلب إصدار PHP >= 7.0

## المثال 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// تشغيل العامل
Worker::runAll();
```

## المثال 2: الاستماع إلى عدة منافذ (بروتوكولات متعددة) باستخدام Workerman

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// بعد تشغيل كل عملية ، أضف استماعًا جديدًا في العملية الحالية
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * عملية تستمع على نفس المنفذ (المأخذ لم يتأصل من العملية الأم)
     * يجب تمكين إعادة استخدام المنفذ، وإلا سيتم الإبلاغ عن خطأ "Address already in use"
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // بدء الاستماع
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// تشغيل العامل
Worker::runAll();
```
