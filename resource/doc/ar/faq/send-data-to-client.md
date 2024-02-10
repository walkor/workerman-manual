يمكنك إرسال بيانات إلى عميل معين باستخدام WorkerMan ، حتى بدون استخدام GatewayWorker.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// بدء وحدة عمل للاستماع على المنفذ 1234
$worker = new Worker('websocket://workerman.net:1234');
// ==== يجب ضبط عدد العمليات هنا إلى واحد ====
$worker->count = 1;
// إضافة خاصية جديدة للحفاظ على تعيينات uid إلى الاتصال
$worker->uidConnections = array();
// عندما يصل عميل ما برسالة
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // التحقق مما إذا كان العميل الحالي قد تم التحقق بالفعل، أي إعداد uid
    if(!isset($connection->uid))
    {
       // إذا لم يتم التحقق يجب أن يكون الحزمة الأولى هو uid (هنا لتوضيح العرض، لم يتم التحقق بالفعل)
       $connection->uid = $data;
       /* الاحتفاظ بتحويلات uid إلى الاتصال، يمكن من هنا البحث بسهولة باستخدام uid لتحقيق توجيه بيانات تستهدف uid معين
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('login success, your uid is ' . $connection->uid);
    }
    // منطق آخر، إما إرسال لuid معين أو اشتراك عام
    // بفرض أن تنسيق الرسالة هو uid: message حينها الرسالة موجهة ل uid
    // عندما يكون uid هو all يكون الاشتراك عام
    list($recv_uid, $message) = explode(':', $data);
    // اشتراك عام
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // إرسال لuid معين
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// عندما يفصل عميل
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // حذف تحويلات عند قطع الاتصال
        unset($worker->uidConnections[$connection->uid]);
    }
};

// إرسال بيانات لكل العملاء المحققين
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// إرسال بيانات لuid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// تشغيل الوحدة العاملة (هذا الإعداد يعرف فقط وحدة واحدة)
Worker::runAll();
```

**ملاحظة:**

يمكنك إرسال بيانات موجهة ل uid في الأمثلة أعلاه، على الرغم من أخذ عملية واحدة ولكن يدعم العمل مع 100000 عميل عبر الإنترنت.
  
يرجى ملاحظة أن هذا المثال يعمل فقط على عملية واحدة، حيث يجب أن يكون $worker->count معبأ 1. إذا كنت ترغب في دعم العمليات المتعددة أو مجموعة خوادم، فيجب استخدام مكون القناة لإتمام التواصل بين العمليات. العملية سهلة للغاية ويمكن الرجوع إلى مثال [Channel Component Cluster Push](../components/channel-examples.md).

**إذا كنت ترغب في إرسال رسالة إلى العميل من نظام آخر، فيرجى الرجوع إلى الفقرة [إرسال من مشروع آخر](push-in-other-project.md)**
