# المثال 1
**``` (يتطلب Workerman الإصدار >= 3.3.0) ```**

نظام الإرسال المتعدد العملية (تجمع موزع) القائم على العامل ، إرسال مجموعة، بث المجموعة.

`start_channel.php`
يمكن أن يتم نشر الخدمة start_channel فقط في النظام بأكمله. يفترض أن يعمل في 192.168.1.1.

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// تهيئة خادم قناة
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
يمكن نشر الخدمة start_ws على عدة خوادم ، يفترض أن تعمل على الخوادم 192.168.1.2 و 192.168.1.3 .
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// خادم الويب سوكيت
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // الاتصال العميل بقناة الخادم
    Channel\Client::connect('192.168.1.1', 2206);
    // باعتبارها معرف عملية ، قم بالاشتراك في الحدث وسجل وظيفة معالجة الأحداث
    $event_name = $worker->id;
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "الاتصال غير موجود\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // الاشتراك في حدث البث
    $event_name = 'بث';
    // عند تلقي حدث بث ، قم بإرسال بيانات البث إلى جميع اتصالات العميل في العملية الحالية
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "معرف العملية:{$worker->id} معرف الاتصال:{$connection->id} تم الاتصال\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
يمكن نشر الخدمة start_http على عدة خوادم، يفترض أن تعمل على الخوادم 192.168.1.4 و 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// يستخدم لمعالجة طلبات http ، لإرسال البيانات إلى أي اتصال عميل ، يجب تمرير معرف العملية ومعرف الاتصال
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // متوافق مع workerman 4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('حسناً');
    if(empty($_GET['content'])) return;
    // إرسال البيانات إلى عملية العمل الفرعي لاتصال العميل المحدد
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // البث العام للبيانات
    else
    {
        $event_name = 'بث';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```


## الاختبار 
1- تشغيل خدمات الخادم على كل خادم.

2- اتصال عملي بالخادم

افتح متصفح Chrome واضغط على F12 لفتح لوحة تحكم التصحيح ، في الجزء Console ، أدخل (أو ضع الكود التالي في صفحة HTML وقم بتشغيله بواسطة JS)

```javascript
// يمكن أيضًا الاتصال ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("تم استلام رسالة من الخادم: " + e.data);
};
```

3- عبر استدعاء واجهة البرنامج النصي HTTP
زيارة عنوان URL ```http://192.168.1.4:4237/?content={$content}``` أو  ```http://192.168.1.5:4237/?content={$content}``` لإرسال البيانات ```$content``` إلى جميع اتصالات العميل.

زيارة عنوان URL ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` أو  ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` لإرسال البيانات ```$content``` إلى اتصال عميل محدد داخل عملية العمل الفرعي.

ملاحظة: عند الاختبار ، قم بتغيير ```{$worker_id}```  ، ```{$connection_id}``` و ```{$content}``` إلى القيم الفعلية.
