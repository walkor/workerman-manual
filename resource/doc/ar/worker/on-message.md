# onMessage
## الوصف:
```php
callback Worker::$onMessage
```

عندما يرسل العميل بيانات عبر الاتصال (عندما يتلقى Workerman البيانات) ، يتم تشغيل التابع الارتجاعي.

## متغيرات التابع الارتجاعي

 ``` $connection ```

كائن الاتصال، وهو [مثيل TcpConnection](../tcp-connection.md) ، يستخدم لعمليات الاتصال بالعميل مثل [إرسال البيانات](../tcp-connection/send.md) ، [اغلاق الاتصال](../tcp-connection/close.md) وغيرها.

 ``` $data ```

البيانات التي تم إرسالها عبر الاتصال من العميل ، إذا كان Worker محدد ببروتوكول معين، فإن $data سيكون البيانات التي تم فك تشفيرها وفقًا للبروتوكول المحدد. نوع البيانات يعتمد على تنفيذ `decode()` للبروتوكول المحدد، مثل `websocket` `text` `frame` سيكون النوع سلسلة أحرف، بينما سيكون البروتوكول HTTP عبارة عن كائن [`Workerman\Protocols\Http\Request`](../http/request.md).


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// تشغيل الووركر
Worker::runAll();
```

ملاحظة: بالإضافة إلى استخدام الدالة المجهولة كوظيفة ارتجاعية، يمكنك أيضًا [الاطلاع هنا](../faq/callback_methods.md) للاطلاع على أساليب الارتجاع الأخرى.
