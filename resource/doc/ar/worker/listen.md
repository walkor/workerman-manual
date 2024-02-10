يستخدم `listen` لتنفيذ الاستماع بعد تطبيق `Worker`.

يستخدم هذا الأسلوب بشكل رئيسي لإنشاء نموذج `Worker` جديد بعد بدء تشغيل عملية `Worker`. يمكنه إمكانية الاستماع إلى عدة منافذ في نفس العملية ويدعم عدة بروتوكولات. يُلاحظ أن استخدام هذا الأسلوب يزيد من الاستماع في العملية الحالية فقط، ولا يؤدي إلى إنشاء عمليات جديدة أو ينشط دالة `onWorkerStart`.

على سبيل المثال، بعد تشغيل `Worker` متخصص في `http`، يمكن إنشاء `websocket Worker` في نفس العملية، مما يتيح للعملية الوصول عبر بروتوكول `http` وأيضًا بروتوكول `websocket`. نظرًا لأن `websocket Worker` و `http Worker` في نفس العملية، فإنها يمكنها الوصول إلى المتغيرات المشتركة ومشاركة جميع اتصالات المأخذ. بإمكانها إستلام طلبات `http` ومن ثم التحكم في عميل `websocket` لإنجاز مثل هذه المهام.

**ملاحظة:**
إذا كانت إصدارة PHP <= 7.0، فلا يدعم إنشاء `Worker` بنفس المنفذ في عمليات فرعية متعددة. على سبيل المثال، إذا قامت العملية A بإنشاء `Worker` للاستماع إلى المنفذ 2016، فإن العملية B لن تتمكن من إنشاء `Worker` والاستماع إلى نفس منفذ 2016، وإلا ستظهر خطأ `Address already in use`. الكود أدناه لا يمكن تشغيله.
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

Worker::runAll();
```

إذا كانت إصدارة PHP>=7.0، يمكنك ضبط `Worker->reusePort=true` لإنشاء `Worker` بنفس المنفذ في عمليات فرعية متعددة. كما هو موضح في الكود التالي:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

Worker::runAll();
```

### مثال على خادم PHP لإرسال الرسائل في الوقت الحقيقي إلى العميل

**المبدأ:**

1. إنشاء `websocket Worker` للحفاظ على اتصالات طويلة مع العملاء.
2. في داخل الـ `websocket Worker`، قم بإنشاء `text Worker`.
3. `websocket Worker` و `text Worker` في نفس العملية، مما يتيح لهم مشاركة اتصالات العميل بسهولة.
4. نظام PHP خلفي مستقل يتواصل مع `text Worker` من خلال بروتوكول النص.
5. `text Worker` يعمل على الاتصال بعملاء `websocket` لإتمام إرسال البيانات.

**الشفرة والخطوات:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:1234');

$worker->count = 1;
$worker->onWorkerStart = function($worker)
{
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        $ret = sendMessageByUid($uid, $buffer);
        $connection->send($ret ? 'ok' : 'fail');
    };
    $inner_text_worker->listen();
};

$worker->uidConnections = array();

$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    if(!isset($connection->uid))
    {
       $connection->uid = $data;
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        unset($worker->uidConnections[$connection->uid]);
    }
};

function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

Worker::runAll();
```

تشغيل خدمة الخلفية
 ```php push.php start -d```

كود js لاستقبال الرسائل الواردة من الخلفية
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

كود إرسال الرسائل من الخلفية
```php
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
$data = array('uid'=>'uid1', 'percent'=>'88%');
fwrite($client, json_encode($data)."\n");
echo fread($client, 8192);
```
