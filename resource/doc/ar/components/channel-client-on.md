```php
# في
**``` (يتطلب Workerman الإصدار 3.3.0 فأعلى) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
اشتراك في حدث ```$event_name``` وتسجيل دالة الاستدعاء ```$callback_function``` عند حدوث الحدث

## معاملات الدالة الاستدعاء

``` $event_name ```

اسم الحدث المشترك فيه، يمكن أن يكون أي سلسلة نصية.

``` $callback_function ```

دالة ارتداء تنشط عند حدوث الحدث. الصيغة الأصلية للدالة هي ```callback_function(mixed $event_data)```. ```$event_data``` هو البيانات التي يتم نشرها عند حدوث الحدث.

تنويه:

إذا تم تسجيل دالتي استدعاء لنفس الحدث، ستحل الدالة الاستدعاء الثانية محل الدالة الاستدعاء الأولى.

## مثال
عامل (Worker) متعدد العمليات (متعدد الخواديم)، عميل واحد يرسل رسالة وتبث لجميع العملاء


start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// بدء سيرفر Channel
$channel_server = new Channel\Server('0.0.0.0', 2206);

// سيرفر websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// عند بدء كل عملية worker
$worker->onWorkerStart = function($worker)
{
    // العميل Channel يتصل بسيرفر Channel
    Channel\Client::connect('127.0.0.1', 2206);
    // الاشتراك في حدث البث (broadcast) وتسجيل دالة استدعاء الحدث
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // نقل رسالة لجميع عمليات الـ worker الحالية
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // استخدام البيانات التي أرسلها العميل كبيانات الحدث
   $event_data = $data;
   // بث حدث البث (broadcast) لجميع عمليات الـ worker
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```


**الاختبار**

افتح متصفح Chrome، ثم اضغط على F12 لفتح لوحة التحكم، في علامة Console قم بكتابة (أو ضع الكود التالي في صفحة HTML ثم قم بتشغيله بواسطة JavaScript)

الاتصال الذي يستقبل الرسائل
```javascript
// قم بتغيير 127.0.0.1 بعنوان Workerman الفعلي
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("تلقيت رسالة من السيرفر: " + e.data);
};
```

بث الرسالة
```
ws.send('مرحبا بالعالم');
```

```
