# الترويس
## الوصف

```php
static bool Worker::$daemonize
```
هذا الخاصية هي خاصية استاتيكية عالمية، وتعني ما إذا كان يتم تشغيل الخادم بوضع الشيطان (daemon) أم لا. إذا استخدمت الأمر البداية المعلمة ```-d```، سيتم ضبط هذه الخاصية تلقائياً على true. ويمكن أيضاً ضبطها يدوياً في الكود.

ملاحظة: يجب ضبط هذه الخاصية قبل تشغيل ```Worker::runAll();``` من أجل أن تكون فعالة. ويتم عدم دعم هذه الميزة في أنظمة ويندوز.

## المثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "بدء تشغيل الخادم\n";
};
// تشغيل الخادم
Worker::runAll();
```
