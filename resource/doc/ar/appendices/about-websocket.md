
# بروتوكول WebSocket

حاليًا، إصدار **بروتوكول WebSocket في Workerman هو 13**.

بروتوكول WebSocket هو بروتوكول جديد في HTML5. يمكنه تحقيق التواصل المزدوج الكامل بين المتصفح والخادم.

## علاقة بروتوكول WebSocket مع TCP

بروتوكول WebSocket وبروتوكول HTTP هما من نفس الطبقة التطبيقية وكلاهما يعتمد على نقل TCP. وبصورة أساسية، بروتوكول WebSocket ذاته لا يرتبط كثيرًا ببروتوكول Socket ولا يمكن مقارنتهما.

## عملية مصافحة بروتوكول WebSocket

بروتوكول WebSocket يشمل عملية مصافحة، حيث يتم التواصل بين المتصفح والخادم باستخدام بروتوكول HTTP أثناء عملية المصافحة في Workerman يمكن التدخل بهذه الطريقة.

**عندما تكون النسخة من workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // يمكن هنا التحقق مما إذا كان مصدر الاتصال مشروع، وإذا كان غير مشروع يمكن إغلاق الاتصال
        // $_SERVER['HTTP_ORIGIN'] تعبر عن الموقع الذي يأتي منه صفحة الويب التي أرسلت طلب اتصال websocket
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // في داخل onWebSocketConnect يمكن استخدام $_GET و $_SERVER
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**عندما تكون النسخة من workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## نقل البيانات الثنائية في بروتوكول WebSocket

بروتوكول WebSocket يمكنه افتراضيًا نقل النصوص UTF8 فقط. إذا كنت بحاجة لنقل بيانات ثنائية، يرجى قراءة الجزء التالي.

في بروتوكول WebSocket، تُستخدم علامة تعريف في رأس البروتوكول للإشارة إلى نوع البيانات المرسلة، ويقوم المتصفح بالتحقق من العلامة واكتشاف ما إذا كان نوع المحتوى المرسل مطابق. إذا كان الأمر غير متطابق، فستتم فصل الاتصال.

لذا، عندما يُرسل البيانات من الخادم، يجب ضبط العلامة التعريفية بناءً على نوع البيانات المُرسلة. في Workerman، إذا كانت البيانات عبارة عن نصوص UTF8 عادية، يجب ضبطها كالتالي (عادةً ما تكون هذه القيمة افتراضية ولا تحتاج إلى ضبطها يدويًا)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

إذا كانت البيانات هي بيانات ثنائية، يجب ضبطها كالتالي
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**ملحوظة**: إذا لم يتم ضبط $connection->websocketType، سيكون القيمة الافتراضية لـ $connection->websocketType هي BINARY_TYPE_BLOB (وهو نوع النصوص UTF8 عادةً). وعمومًا، يتم نقل بيانات النصوص UTF8 في التطبيقات، مثل نقل بيانات JSON، لذا لا يجب ضبط $connection->websocketType يدويًا. فقط عند نقل بيانات ثنائية (مثل بيانات الصور أو بروتوبافر وما إلى ذلك) يجب ضبط هذا الخصائص كـ BINARY_TYPE_ARRAYBUFFER.

## استخدام Workerman كعميل WebSocket
يمكنك استخدام فئة [AsyncTcpConnection](../async-tcp-connection.md) بالاشتراك مع [بروتوكول ws](about-ws.md) لاستخدام Workerman كعميل WebSocket للاتصال بخادم WebSocket عن بعد وإكمال التواصل المزدوج في الوقت الفعلي.
