البروتوكول ws

في الوقت الحالي، نسخة **ws لبروتوكول workerman هي 13**.

يمكن لـ workerman أن يعمل كعميل من خلال بروتوكول ws، عن طريق إجراء اتصال websocket، مع الخادم البعيد، لتحقيق التواصل ذو الاتجاهين.

> **ملحوظة**
> البروتوكول ws يمكن استخدامه فقط كعميل من خلال AsyncTcpConnection، ولا يمكن استخدامه كبروتوكول للإستماع على الخادم كبروتوكول websocket. وبمعنى آخر، الكتابة التالية غير صحيحة.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

إذا كنت ترغب في استخدام workerman كخادم websocket، يرجى استخدام [بروتوكول websocket](about-websocket.md).

**مثال على استخدام ws كبروتوكول عميل websocket:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// عند بدء التشغيل
$worker->onWorkerStart = function()
{
    // تواصل مع خادم websocket البعيد ببروتوكول websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // أرسل قلب حيوي websocket كل 55 ثانية إلى الخادم (اختياري)
    $ws_connection->websocketPingInterval = 55;
    // قم بتعيين الرؤوس HTTP (اختياري)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // قم بتعيين نوع البيانات (اختياري)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB للنصوص و BINARY_TYPE_ARRAYBUFFER للبيانات الثنائية
    // عندما يكتمل TCP ثلاث مرات اتصال (اختياري)
    $ws_connection->onConnect = function($connection){
        echo "tcp متصل \n";
    };
    // عند الانتهاء من مصافحة websocket (اختياري)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('مرحبا');
    };
    // عندما يتلقى خادم websocket البعيد رسالة
    $ws_connection->onMessage = function($connection, $data){
        echo "الاستلام: $data\n";
    };
    // عند حدوث خطأ في الاتصال، عادةً ما يحدث خطأ في الاتصال بخادم websocket بعيد (اختياري)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "خطأ: $msg\n";
    };
    // عند فصل الاتصال بخادم websocket البعيد (اختياري، يُوصى بإضافة إعادة الاتصال)
    $ws_connection->onClose = function($connection){
        echo "تم إغلاق الاتصال ومحاولة إعادة الاتصال\n";
        // إذا تم فصل الاتصال، قم بإعادة الاتصال بعد ثانية واحدة
        $connection->reConnect(1);
    };
    // بعد تعيين جميع التوابع المرتبطة بهذه الدعوات، قم بإجراء عملية الاتصال
    $ws_connection->connect();
};
Worker::runAll();
```

للمزيد، يُرجى الرجوع إلى [كونه عميل لبروتوكول ws/wss](../faq/as-wss-client.md).
