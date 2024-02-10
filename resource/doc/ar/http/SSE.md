SSE 
**هذه الميزة تتطلب workerman >= 4.0.0**

SSE أو Server-sent Events هي تقنية لدفع الخادم. جوهرها هو أن العميل يرسل طلب HTTP يحمل رأس `Accept: text/event-stream`، ثم يظل الاتصال مفتوحًا ويمكن للخادم أن يواصل دفع البيانات إلى العميل على هذا الاتصال.

الفرق بينها وبين websocket هو:
*   SSE يمكن فقط للخادم أن يدفع بيانات للعميل؛ WebSocket يمكنه التواصل في الاتجاهين.
*   SSE يدعم الإعادة التلقائية للاتصال بشكل افتراضي؛ WebSocket يتطلب تنفيذ ذلك يدويًا.
*   SSE يمكنه فقط نقل النصوص UTF-8، والبيانات الثنائية يجب ترميزها إلى UTF-8 قبل النقل؛ WebSocket يدعم نقل البيانات UTF-8 والبيانات الثنائية بشكل افتراضي.
*   SSE يحتوي على أنواع الرسائل بشكل افتراضي؛ WebSocket يتطلب تنفيذ ذلك يدويًا.

### مثال
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // إذا كان رأس الطلب هو text/event-stream فهذا يعني طلب SSE
    if ($request->header('accept') === 'text/event-stream') {
        // أولاً، أرسل استجابة تحتوي على رأس Content-Type: text/event-stream
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // ادفع البيانات بانتظام إلى العميل
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // عند إغلاق الاتصال يجب حذف المؤقت لتجنب تراكمه وتسرب الذاكرة
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // أرسل حدث message يحمل البيانات hello ولا يلزم تحديد معرف الرسالة
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// قم بتشغيل الخادم
Worker::runAll();
```

كود العميل JavaScript
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // سيتم طباعة الكلمة hello
}, false);
```
