# onConnect
## الوصف:
```php
callback Worker::$onConnect
```

عندما يقوم العميل بتأسيس اتصال مع Workerman (بعد إكمال مصافحة TCP الثلاثية)، يتم تنشيط الدالة الردية. سيتم تنشيط دالة `onConnect` مرة واحدة فقط لكل اتصال.

ملاحظة: حدث onConnect يمثل فقط إكمال مصافحة TCP بين العميل و Workerman، وفي هذا الوقت لم يتم بعد إرسال أي بيانات من العميل. لذا، باستثناء الحصول على عنوان IP البعيد باستخدام `$connection->getRemoteIp()`، لا توجد بيانات أو معلومات أخرى يمكن استخدامها لتحديد هوية العميل في حدث onConnect. إذا أردت معرفة هوية العميل، يجب على العميل إرسال بيانات توثيق، مثل رمز مميز معين أو اسم مستخدم وكلمة مرور، ومن ثم إجراء التوثيق في [استجابة onMessage](on-message.md).

نظرًا لأن UDP لا يعتمد على الاتصال، فإن استخدام بروتوكول UDP لن ينشط حدث onConnect ولن ينشط حدث onClose.

## معاملات الدالة الردية

 ``` $connection ```

كائن الاتصال، أي [مثيل TcpConnection](../tcp-connection.md)، والذي يستخدم للتعامل مع اتصال العميل، مثل [إرسال البيانات](../tcp-connection/send.md)، [إغلاق الاتصال](../tcp-connection/close.md) وما إلى ذلك.

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "اتصال جديد من عنوان IP " . $connection->getRemoteIp() . "\n";
};
// تشغيل الوركر
Worker::runAll();
```

تلميح: بالإضافة إلى استخدام الدالة الخفية كدالة ردية، يمكنك [الرجوع إلى هنا](../faq/callback_methods.md) لاستخدام أساليب ردية أخرى.
