# الإغلاق
## الوصف:
```php
callback Worker::$onClose
```

هو الاستدعاء الردود الفعلية التي تُشغل عند فصل العميل عن Workerman. بغض النظر عن كيفية فصل الاتصال، سيتم تنفيذ $onClose عند الافصاح عن الاتصال. سيتم تنفيذ $onClose مرة واحدة فقط لكل اتصال.

ملاحظة: إذا تم فصل الاتصال بسبب انقطاع الإنترنت أو الكهرباء أو حالات قصوى أخرى، فيحتاج Workerman إلى الهامش الزمني لإرسال حزمة fin tcp إلى العميل وبالتالي فإن Workerman لن يعرف بشكل فوري بانقطاع الاتصال وبالتالي لن يتم تنفيذ $onClose بشكل فوري. يتطلب حالات كهربائيـة القصوى هذه حلولًا تتعلق بالتأشيرة عند المستوى التطبيقي. تصلح حلول الإشارات القلبية عندما يحتاجها Workerman، قم بالرجوع إلى تنفيذ إشارات القلبية في Workerman [هنا](../faq/heartbeat.md). إذا كنت تستخدم إطار GatewayWorker، يمكنك استخدام آلية إشارات القلب في إطار GatewayWorker مباشرة، راجع [هنا](https://doc2.workerman.net/heartbeat.html).

نظرًا لأن بروتوكول نقل البيانات المتزامن (UDP) لا يصادق اتصالات، فإنه عند استخدام UDP، لن يتم تنفيذ استدعاء الاقتران (onConnect) أو استدعاء الإغلاق (onClose).

## معلمات الاستدعاء الردود الفعلية
``` $connection ```

كائن الاتصال، أي [مثيل TcpConnection](../tcp-connection.md)، المستخدم لتشغيل اتصال العميل، مثل [إرسال البيانات](../tcp-connection/send.md)، [إغلاق الاتصال](../tcp-connection/close.md) وما إلى ذلك.


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "تم إغلاق الاتصال\n";
};
// تشغيل الوركر
Worker::runAll();
```

ملاحظة: بالإضافة إلى استخدام الدوال المجهولة لتنفيذ الردود الفعلية، يمكن استخدام أوضاع ردود الفعل الأخرى [تفضل بزيارة هذا الرابط](../faq/callback_methods.md).
