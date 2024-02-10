# onError
## الوصف:
```php
callback Worker::$onError
```

عند حدوث خطأ في اتصال العميل، يتم تشغيل هذا الدالة.

الأنواع الحالية للأخطاء:

1. فشل في استدعاء Connection::send بسبب فصل اتصال العميل (سيتم تشغيل دالة onClose بعد ذلك) 
```(code:WORKERMAN_SEND_FAIL msg:client closed)```

2. بعد تشغيل onBufferFull (حيث يكون الطرف الآخر ممتلئ)، يتم استدعاء Connection::send مرة أخرى ويكون الطرف الآخر ممتلئ ممّا يؤدي إلى فشل الإرسال (لن يتم تشغيل دالة onClose)
```(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3. عند فشل AsyncTcpConnection في الاتصال (سيتم تشغيل دالة onClose بعد ذلك) 
```(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client returns the error message)```

## معاملات الدالة الردية
``` $connection ```

كائن الاتصال، أي [نموذج TcpConnection](../tcp-connection.md)، يستخدم لعمليات التحكم بالاتصال بالعميل مثل [إرسال البيانات](../tcp-connection/send.md)، [إغلاق الاتصال](../tcp-connection/close.md) وغيرها

``` $code ```

كود الخطأ

``` $msg ```

رسالة الخطأ

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// تشغيل الوركر
Worker::runAll();
```

ملحوظة: بالإضافة إلى استخدام الدالة المجهولة كدالة ردية، يمكنك أيضًا [الرجوع إلى هنا](../faq/callback_methods.md) لاستخدام طرق ردية أخرى.
