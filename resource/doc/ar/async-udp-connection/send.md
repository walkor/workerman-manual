الطريقة send

```php
void AsyncUdpConnection::send(string $data)
```
ينفذ عملية الاتصال الغير متزامنة. تقوم هذه الطريقة بالعودة فورًا.

### المعاملات
```$data```
البيانات المُرسَلة إلى الخادم، وحجم البيانات لا يمكن أن يتجاوز 65507 بايت (أقصى حجم لنقل حزمة بيانات واحدة في البروتوكول UDP هو 65507 بايت)، وإلا سيفشل الإرسال.

### القيمة المُعادة
ليس لها قيمة معادة.

### مثال

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // بعد ثانية واحدة، قم بتشغيل عميل UDP والاتصال بالمنفذ 1234 وإرسال السلسلة "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // تلقى بيانات الخادم "hello"
            echo "recv $data\r\n";
            // إغلاق الاتصال
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // تلقى بيانات من عميل AsyncUdpConnection، ويقوم بإرسال السلسلة "hello"
    $connection->send("hello");
};
Worker::runAll();
```

بعد التنفيذ، سيظهر ما يلي:
```
recv hello
```
