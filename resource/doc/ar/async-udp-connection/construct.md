# الطريقة __construct

```php
void AsyncUdpConnection::__construct(string $remote_address)
```
يُنشئ كائن اتصال UDP.

يمكن لـ AsyncUdpConnection أن يتيح لـ Workerman كونه عميلًا يرسل بيانات UDP إلى الخادم البعيد.

## المُعاملات
المُعامل: ``` remote_address ```

عنوان الاتصال، على سبيل المثال
``` udp://192.168.1.1:1234 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

## مثال

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // بعد ثانية واحدة، يتم تشغيل عميل UDP، يتصل بالمنفذ 1234 ويُرسل سلسلة نصية "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // يتم استلام البيانات "hello" من الخادم
            echo "recv $data\r\n";
            // يُغلق الاتصال
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // يُستلم بيانات العميل AsyncUdpConnection ويتم إرجاع سلسلة النص "hello"
    $connection->send("hello");
};
Worker::runAll();     
```
عند التشغيل، يُطبع شيء مُشابه للتالي:
```
recv hello
```
