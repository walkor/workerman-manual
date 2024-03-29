# النبض (القلب)

ملاحظة: التطبيقات ذات الاتصال الطويل يجب أن تكون لها نبضات قلب، وإلا فإن الاتصال قد يتم فصله بشكل قاطع من قبل نقاط التوجيه بسبب عدم التواصل لفترة طويلة.

يوجد دور أساسيين لنبضات القلب:

1. يقوم العميل بإرسال بيانات بانتظام إلى الخادم لمنع انقطاع الاتصال نتيجة لإغلاق بعض جدران الحماية على الشبكة بسبب انقطاع التواصل لفترة طويلة.
2. يمكن للخادم استخدام نبضات القلب للكشف عما إذا كان العميل متصلاً. إذا لم يصل العميل بأي بيانات خلال الفترة المحددة، سيتم اعتباره متصلاً. وبهذه الطريقة يمكن اكتشاف انقطاع الاتصال بسبب حالات نادرة (انقطاع التيار الكهربائي، انقطاع الاتصال بالشبكة، إلخ).

يُنصح بفاصل زمني لنبضات القلب: 

يُوصى بأن يكون فاصل زمني لإرسال نبضات القلب عند العميل أقل من 60 ثانية، مثل 55 ثانية.

> لا توجد متطلبات محددة لتنسيق بيانات نبضات القلب، المهم أن يمكن للخادم التعرف عليها.

## مثال على نبضات القلب
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// فاصل زمني لنبضات القلب 55 ثانية
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // قم بتعيين خاصية lastMessageTime للاتصال مؤقتًا لتسجيل وقت استقبال الرسالة الأخيرة
    $connection->lastMessageTime = time();
    // وظائف الأعمال الأخرى...
};

// عند بدء التشغيل، قم بتعيين موقت يعمل كل 10 ثوانٍ
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function()use($worker){
        $time_now = time();
        foreach($worker->connections as $connection) {
            // قد لا يكون هذا الاتصال قد استقبل رسالة حتى الآن، لذا قم بتعيين lastMessageTime بالوقت الحالي
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // إذا كانت مدة التواصل السابق أكبر من فاصل نبضات القلب، تعتبر أن العميل قد انقطع، فأغلق الاتصال
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

يعتبر الضبط أعلاه أنه إذا لم يرسل العميل أي بيانات للخادم لأكثر من 55 ثانية، سيعتبر الخادم أن العميل قد انقطع وسيغلق الاتصال ويحدث الحدث onClose.

## إعادة الاتصال بعد الانقطاع (مهم)

سواء كان العميل يرسل نبضات القلب أو الخادم يرسل نبضات القلب، فإنه من الممكن قطع الاتصال. على سبيل المثال، عند تصغير نافذة المتصفح يتم إيقاف تشغيل جافا سكريبت، أو عند التبديل إلى صفحة علامة تبويب أخرى يتم إيقاف تشغيل جافا سكريبت، أو عند دخول الكمبيوتر في وضع السكون، وما إلى ذلك، أو عند تغيير شبكة الجوال أو ضعف الإشارة، أو إغلاق الشاشة السوداء للهاتف، أو تبديل تطبيق الهاتف إلى الخلفية، أو فشل في الاتصال بالشبكة، أو قطع الخادم عن الاتصال بشكل نشط، إلخ. ولا سيما في بيئة الإنترنت الخارجية المعقدة، فإن العديد من نقاط التوجيه قد تقوم بتنظيف الاتصالات غير النشطة في غضون دقيقة واحدة، وهذا هو السبب في توصية بفوارق زمنية لنبضات القلب تكون أقل من دقيقة.

يُعتبر فصل الاتصال بسبب تعقيدات الإنترنت خارج الشبكة سهلاً، لذا فإن إعادة الاتصال بعد الانقطاع هي وظيفة ضرورية يجب أن تكون متوفرة في تطبيقات الاتصال الطويل (إن إعادة الاتصال بعد الانقطاع يمكن أن تتم فقط على العميل، لا يمكن تنفيذها على الخادم). على سبيل المثال، في حالة websocket في المتصفح، ينبغي أن يكون هناك استماع لحدث onclose، وعندما يحدث حدث onclose يجب إنشاء اتصال جديد (لتجنب حاجة إلى إعادة تشغيل).
على نحو أشد، ينبغي للخادم أيضًا أن يقوم بإرسال بيانات النبضات القلب بانتظام، ويجب على العميل مراقبة بيانات النبضات القلب المرسلة من قبل الخادم ومراقبة ما إذا كان الوقت المنصرف للبيانات زائدًا عن الحد المسموح به. إذا لم يتم استلام بيانات النبضات القلب من الخادم في الوقت المنصرف، فيجب اعتبار الاتصال قد انقطع وتنفيذ الأمر close لإغلاق الاتصال وإنشاء اتصال جديد.
