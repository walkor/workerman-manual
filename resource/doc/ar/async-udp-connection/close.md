```php
void Connection::close(mixed $data = '')
```
أغلق الاتصال بشكل آمن وقم بتشغيل callback ```onClose``` للاتصال.

على الرغم من أن بروتوكول UDP ليس له اتصال، إلا أن كائن AsyncUdpConnection المقابل يتم الاحتفاظ به دائمًا في الذاكرة، ويجب استدعاء الدالة close لتحرير كائن الاتصال UDP المقابل، وإلا سيتم الاحتفاظ بهذا الكائن في الذاكرة باستمرار، مما يؤدي إلى تسرب الذاكرة.

## المعاملات

``` $data ```

معلمة اختيارية، البيانات التي سيتم إرسالها (إذا كان هناك بروتوكول محدد، سيتم استدعاء دالة ترميز البروتوكول تلقائيًا لتغليف بيانات ```$data``` )، عند الانتهاء من إرسال البيانات، سيتم إغلاق الاتصال، وبعد ذلك سيتم تشغيل callback onClose.

لا يمكن أن يتجاوز حجم البيانات 65507 بايت، وإلا ستفشل عملية الإرسال.

### مثال

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // بعد ثانية واحدة، قم بتشغيل عميل UDP، والاتصال بالمنفذ 1234 وإرسال سلسلة hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // استقبال البيانات المرجعية من الخادم hello
            echo "recv $data\r\n";
            // أغلق الاتصال
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // تلقي بيانات من عميل AsyncUdpConnection المرسلة، وإرجاع سلسلة hello
    $connection->send("hello");
};
Worker::runAll();             
```

سيتم طباعة ما يشبه التالي بعد التنفيذ:
``` 
recv hello
```
