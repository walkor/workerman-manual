# onBufferDrain
## الوصف:
```php
callback Worker::$onBufferDrain
```

كل اتصال لديه مساحة تخزين مؤقت لإرسال الطبقة التطبيقية، حجم المساحة التخزينية يتم تحديده بواسطة ```TcpConnection::$maxSendBufferSize``` والقيمة الافتراضية هي 1 ميجابايت، يمكنك ضبط القيمة يدوياً وسيكون له تأثير على جميع الاتصالات. 

سيتم تشغيل هذا التابع عندما يتم إرسال جميع بيانات المساحة التخزينية لإرسال الطبقة التطبيقية. عادة ما يتم استخدامه بالتزامن مع onBufferFull، على سبيل المثال، في حالة onBufferFull سيتم إيقاف إرسال البيانات إلى الطرف الآخر، ثم عند الonBufferDrain يتم استئناف إرسال البيانات.


## معلمات دالة الرد
``` $connection ```

كائن الاتصال، بمعنى [مثيل TcpConnection](../tcp-connection.md)، يستخدم للتعامل مع اتصال العميل مثل [إرسال البيانات](../tcp-connection/send.md)، [إغلاق الاتصال](../tcp-connection/close.md) وما إلى ذلك.


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "المساحة التخزينية ممتلئة ولن يتم إرسال المزيد\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "المساحة التخزينية فارغة وسيتم استئناف الإرسال\n";
};
// تشغيل العامل
Worker::runAll();
```

ملحوظة: بالإضافة إلى استخدام الوظيفة المجهولة كدالة رد، يمكنك أيضًا [الرجوع إلى هنا](../faq/callback_methods.md) لاستخدام طرق دالة رد أخرى.

## انظر أيضًا
onBufferFull: يتم تشغيله عندما تمتلئ المساحة التخزينية لإرسال الطبقة التطبيقية للاتصال.
