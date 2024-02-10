# onBufferFull
## الوصف:

```php
callback Worker::$onBufferFull
```

لكل اتصال هناك مساحة تخزين مخصصة لإرسال البيانات على مستوى التطبيق، إذا كانت سرعة استقبال البيانات من العميل أبطأ من سرعة إرسال البيانات من الخادم، فإن البيانات ستخزن مؤقتًا في مساحة التخزين على مستوى التطبيق، وإذا امتلأت هذه المساحة، ستؤدي إلى استدعاء callback `onBufferFull`.

حجم المساحة هو [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)، والقيمة الافتراضية هي 1 ميغابايت، ويمكن تعيين حجم مساحة التخزين بشكل ديناميكي للاتصال الحالي مثل:
```php
// تعيين حجم مساحة التخزين للإرسال للاتصال الحالي، بالبايت
$connection->maxSendBufferSize = 102400;
```
يمكن أيضًا استخدام [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md) لتعيين حجم افتراضي لمساحة التخزين لجميع الاتصالات، مثال على الكود:
```php
use Workerman\Connection\TcpConnection;
// تعيين الحجم الافتراضي لمساحة تخزين الإرسال على مستوى التطبيق لجميع الاتصالات، بالبايت
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

قد يتم استدعاء هذا الcallback **ربما** فور الاتصال بالدالة Connection::send, على سبيل المثال، عندما يتم إرسال بيانات كبيرة أو عندما يتم إرسال البيانات بشكل متتابع بسرعة إلى الطرف الآخر، بسبب الشبكة أو أسباب أخرى يتم تراكم البيانات في مساحة تخزين الإرسال المناسبة للاتصال. وعندما تتجاوز هذه البيانات الحد الأقصى لـ `TcpConnection::$maxSendBufferSize` فإنه يتم استدعاء callback `onBufferFull`.

عند حدوث حدث onBufferFull، عادة ما يحتاج المطور إلى اتخاذ إجراءات مثل التوقف عن إرسال البيانات إلى الطرف الآخر، والانتظار حتى يتم إرسال البيانات في مساحة التخزين (حدث onBufferDrain) على سبيل المثال.

عند استدعاء Connection::send(`$A`) ويؤدي إلى استدعاء onBufferFull، بغض النظر عن كمية البيانات `$A` التي تم إرسالها، حتى لو تجاوزت حجم `TcpConnection::$maxSendBufferSize`، فإن البيانات المراد إرسالها ستضاف إلى مساحة التخزين للإرسال. وهذا يعني أن البيانات التي تم وضعها في مساحة التخزين للإرسال قد تكون أكبر بكثير من `TcpConnection::$maxSendBufferSize`. وعندما تكون البيانات في مساحة التخزين أكبر من `TcpConnection::$maxSendBufferSize` فإن استدعاء Connection::send(`$B`) سيؤدي إلى عدم تضمين البيانات `$B` في مساحة التخزين، وبدلاً من ذلك سيتم تجاهلها واستدعاء `onError` callback.

باختصار، طالما أن مساحة التخزين للإرسال لم تمتلئ، حتى لو كان هناك مساحة خالية حتى بالبايت الواحد، فإن استدعاء Connection::send(`$A`) سيقوم بتضمين `$A` في مساحة التخزين للإرسال بالتأكيد، وإذا تم وضع البيانات في مساحة التخزين للإرسال وكان حجمها يتجاوز الحد الأقصى المسموح به في `TcpConnection::$maxSendBufferSize` فإنه سيتم استدعاء callback `onBufferFull`.


## معاملات الدالة الردود:
``` $connection ```

هو كائن الاتصال، أي [TcpConnection instance](../tcp-connection.md)، المستخدم للتعامل مع اتصال العميل، مثل [إرسال البيانات](../tcp-connection/send.md)، [إغلاق الاتصال](../tcp-connection/close.md) وما إلى ذلك.


## مثال:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
// تشغيل الوركر
Worker::runAll();
```


ملحوظة: بالإضافة إلى استخدام الدالة المجهولة كدالة رد فعل، يمكنك أيضًا [الاطلاع على هذا](../faq/callback_methods.md) لمعرفة كيفية كتابة الدوال الأخرى كدوال رد فعل.


## انظر أيضًا:
onBufferDrain عندما تكون جميع البيانات في مساحة التخزين للإرسال في الاتصال تم إرسالها على مية.
