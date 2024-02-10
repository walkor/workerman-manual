# الدالة connect

```php
void AsyncUdpConnection::connect()
```
تنفيذ عملية الاتصال الغير متزامنة. ستعود هذه الدالة فوراً.

### المعلمات
لا توجد معلمات

### القيمة المُرجعة
لا توجد قيمة مُرجعة

### مثال

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // بعد ثانية واحدة، قم بتشغيل عميل udp، والاتصال بالمنفذ 1234 وإرسال سلسلة نصية "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // استقبال بيانات مرسلة من الخادم "hello"
            echo "recv $data\r\n";
            // إغلاق الاتصال
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // استقبال البيانات المرسلة من عميل AsyncUdpConnection، وإرجاع سلسلة نصية "hello"
    $connection->send("hello");
};
Worker::runAll();         
```

بعد التنفيذ، سيتم طباعة شيء مماثل للتالي:
```php
recv hello
```
