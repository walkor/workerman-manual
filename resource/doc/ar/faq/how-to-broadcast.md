كيفية بث البيانات

## مثال (بث مجدول)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// في هذا المثال ، يجب أن يكون عدد العمليات 1
$worker->count = 1;
// عند بدء العملية ، قم بتعيين موقت لإرسال البيانات إلى جميع اتصالات العميل بانتظام
$worker->onWorkerStart = function($worker)
{
    // بانتظام ، كل 10 ثوانٍ
    Timer::add(10, function()use($worker)
    {
        // تمرير كل اتصال لعميل في العملية الحالية ، وإرسال وقت الخادم الحالي
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// تشغيل العامل
Worker::runAll();
```

## مثال (محادثة جماعية)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// في هذا المثال ، يجب أن يكون عدد العمليات 1
$worker->count = 1;
// عندما يرسل العميل رسالة ، قم ببثها للمستخدمين الآخرين
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// تشغيل العامل
Worker::runAll();
```

## توضيح:
**عملية واحدة:**
الأمثلة أعلاه يمكن أن تعمل فقط مع **عملية واحدة** (```$worker->count=1```)، لأن عند الاستخدام مع عمليات متعددة ، يمكن للعميل الاتصال بعمليات مختلفة وبالتالي فإن العملاء بين العمليات معزولون ولا يمكن التواصل مباشرة بينهم. (لتحقيق هذا ، يجب الاتصال بين العمليات من خلال التواصل بين العمليات ، على سبيل المثال يمكن استخدام مكون القناة مثل [مثال-إرسال عبر الشبكة](../components/channel-examples.md) ،[مثال-الإرسال الجماعي](../components/channel-examples2.md)).

**نوصي باستخدام GatewayWorker**
توفر إطار GatewayWoker الذي تم تطويره بناءً على workerman آلية سهلة للإرسال ، بما في ذلك البث والتوزيع ، ويمكن نشره على عدة عمليات أو حتى عدة خوادم. إذا كنت بحاجة إلى إرسال البيانات إلى العميل ، فإننا نوصي باستخدام إطار GatewayWorker.

عنوان دليل GatewayWorker https://doc2.workerman.net
