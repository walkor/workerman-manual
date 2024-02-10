# تخزين الكائنات والموارد بشكل دائم
في تطوير الويب التقليدي، يتم إطلاق العنان لجميع الكائنات، البيانات، والموارد التي أنشأها PHP بمجرد انتهاء الطلب، مما يجعل من الصعب تحقيق الاستمرارية. ومع ذلك، يمكن بسهولة تحقيق ذلك في WorkerMan.

يمكنك في WorkerMan تخزين البيانات أو الموارد بشكل دائم في الذاكرة عن طريق إضافتها إلى متغيرات عالمية أو أعضاء ثابتة في الفئة.

على سبيل المثال، في الكود أدناه:

يتم استخدام المتغير العالمي `$connection_count` لتخزين عدد اتصالات العملاء في العملية الحالية.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// متغير عالمي لتخزين عدد اتصالات العملاء في العملية الحالية
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // عندما يحدث اتصال جديد، يتم زيادة عدد الاتصالات بواحد
    global $connection_count;
    ++$connection_count;
    echo "الآن عدد الاتصالات=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // عند إغلاق العميل، يتم زيادة عدد الاتصالات بواحد
    global $connection_count;
    $connection_count--;
    echo "الآن عدد الاتصالات=$connection_count\n";
};

```

## انظر إلى نطاق المتغيرات في PHP:
https://php.net/manual/zh/language.variables.scope.php
