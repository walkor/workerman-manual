
# الوظيفة reConnect

```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

 ``` (تتطلب إصدار Workerman >= 3.3.5) ```

إعادة الاتصال. عادة ما يتم استدعاءها في تابع `onClose` لإعادة الاتصال بعد فقدان الاتصال.

يمكن استدعاء هذه الوظيفة لإعادة الاتصال في حالة فقدان الاتصال بسبب مشكلات الشبكة أو إعادة تشغيل خدمة الطرف الآخر.

### المعاملات
``` $delay ```

الوقت الذي يجب أن يتأخر فيه إعادة الاتصال. الوحدة بالثواني ويدعم الأعداد العشرية ويمكن تحديده بدقة إلى الجزء الألفي من الثانية.

إذا لم يتم تمرير المعلمة أو كانت قيمتها 0 ، فإن ذلك يعني الاتصال الفوري.

من الأفضل تمرير معلمة تأخير إعادة الاتصال لتأخير التنفيذ ، وذلك لتجنب استهلاك الوحدة المعالجة المركزية بشكل مفرط في حالة عدم القدرة على الاتصال بسبب مشكلة في خادم الطرف الآخر.

### القيمة المُرجَعَة
لا توجد قيمة مُرجَعَة

### مثال

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // في حالة فقدان الاتصال ، قم بإعادة الاتصال بعد ثانية واحدة
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **ملاحظة**
> بعد نجاح الإعادة التلقائية للاتصال، سيتم استدعاء تابع onConnect للكائن $con مرة أخرى (إذا كان معيناً). في بعض الأحيان نريد أن يتم تنفيذ تابع onConnect مرة واحدة فقط بعد الاتصال الأول، بدون تنفيذه أثناء عملية إعادة الاتصال. يمكن الرجوع إلى المثال التالي:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // في حالة فقدان الاتصال ، قم بإعادة الاتصال بعد ثانية واحدة
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
