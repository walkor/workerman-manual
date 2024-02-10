يمكن إنشاء عمليات مهمات مسبقًا على نفس الجهاز أو على خوادم أخرى أو حتى مجموعة خوادم لمعالجة الأعمال الشاقة. يمكن فتح عدد أكبر من عمليات المهام ، مثل 10 أضعاف وحدة المعالجة المركزية ، ومن ثم يمكن للطرف الخارجي استخدام AsyncTcpConnection لإرسال البيانات بشكل غير متزامن إلى عمليات المهام هذه للمعالجة الغير متزامنة والحصول على نتائج المعالجة الغير متزامنة. 

الخادم العامل عمليات المهام
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// عمال المهام ، باستخدام بروتوكول النص
$task_worker = new Worker('Text://0.0.0.0:12345');
// يمكن فتح عدد أكبر من عمليات المهام حسب الحاجة
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // نفترض أن البيانات المرسلة هي بتنسيق json
     $task_data = json_decode($task_data, true);
     // يتم التعامل مع task_data حسب المنطقية المناسبة .... للحصول على النتيجة ، يتم تجاهل الخطوة هنا ...
     $task_result = ......
     // إرسال النتيجة
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

الاستدعاء في workerman
```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// خدمة ويب الفيب
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // إنشاء اتصال غير متزامن مع خدمة المهام البعيدة ، عنوان IP هو عنوان خدمة المهام البعيدة ، إذا كان على نفس الجهاز فإنه 127.0.0.1 ، إذا كان على مجموعة خوادم فهو عنوان LVS
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // البيانات والمعلمات للمهمة
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // إرسال البيانات
    $task_connection->send(json_encode($task_data));
    // الحصول على النتائج بطريقة غير متزامنة
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // النتيجة
         var_dump($task_result);
         // بمجرد الحصول على النتيجة ، تأكد من إغلاق الاتصال غير المتزامن
         $task_connection->close();
         // إعلام عميل ويب المتصل بأن المهمة اكتملت
         $ws_connection->send('task complete');
    };
    // تنفيذ الاتصال الغير متزامن
    $task_connection->connect();
};

Worker::runAll();
```

بهذه الطريقة ، تتولى عمليات المهام الشاقة المعالجة على نفس الجهاز أو على خوادم أخرى ، وبعد الانتهاء من المهمة ، سيتم الحصول على النتيجة بشكل غير متزامن وبالتالي لن يتم حظر عمليات الأعمال.
