# كـعميل ws/wss

في بعض الأحيان قد نحتاج لاستخدام workerman كعميل للاتصال بخادم معين باستخدام بروتوكول ws/wss والتفاعل معه. فيما يلي مثال لذلك.

## Workerman كعميل ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // عند نجاح مصافحة websocket
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // عند استلام رسالة
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman كعميل wss (ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // SSL يجب الوصول إلى النقطة 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // ضبط الوصول بواسطة ssl لجعلها wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman كعميل wss (ws+ssl) بشهادة SSL محلية

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ضبط أي بي المضيف المحلي للوصول والمنفذ مع الشهادة ssl
    $context_option = array(
        'ssl' => array(
            'local_cert'        => '/your/path/to/pemfile', // مسار الشهادة المحلية
            'passphrase'        => 'your_pem_passphrase', // كلمة مرور ملف PEM
            'allow_self_signed' => true, // السماح بشهادة غير موقعة
            'verify_peer'       => false // السماح بالتحقق من شهادة SSL
        )
    );

    // SSL يجب الوصول إلى النقطة 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // ضبط الوصول بواسطة ssl لجعلها wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## إعدادات أخرى
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();

$worker->onWorkerStart = function()
{
    // الاتصال بخادم websocket البعيد باستخدام بروتوكول websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // إرسال نبضات websocket كل 55 ثانية إلى الخادم برمجية 0x9
    $ws_connection->websocketPingInterval = 55;
    // رؤوس HTTP مخصصة
    $ws_connection->headers = ['token' => 'value'];
    // ضبط نوع البيانات الافتراضي إلى BINARY_TYPE_BLOB للنصوص
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; 
    // عند الاتصال نهائيًا بTCP
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // عند الاتصال نهائيًا بـ websocket
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // عند تلقي رسالة من خادم websocket البعيد
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // في حالة حدوث خطأ مثل عدم قدرة الاتصال بخادم websocket البعيد
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // عند قطع الاتصال بخادم websocket البعيد
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // في حالة فصل الاتصال، أعد التوصيل بعد ثانية واحدة
        $connection->reConnect(1);
    };
    // بعد ضبط كافة الدعوات المذكورة أعلاه، قم بتنفيذ عملية الاتصال
    $ws_connection->connect();
};
Worker::runAll();
```
