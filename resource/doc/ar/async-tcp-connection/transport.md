# الخاصية النقل (transport)

```(workerman >= 3.3.4)```

تعيين خاصية النقل، القيم الممكنة هي [tcp](https://baike.baidu.com/subview/32754/8048820.htm) و [ssl](https://baike.baidu.com/view/525499.htm)، والقيمة الافتراضية هي tcp.

عندما تكون القيمة الخاصية هي [ssl](https://baike.baidu.com/view/525499.htm)، يجب أن يكون PHP مثبتًا [امتداد openssl ](https://php.net/manual/zh/book.openssl.php).

عندما يكون Workerman كعميل يقوم بإرسال طلب اتصال مشفر ssl إلى خادم (اتصال https ، اتصال wss ، إلخ) ، يرجى تعيين هذا الخيار إلى ```ssl```، مثال يظهر ذلك أدناه.

### مثال (اتصال https)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// عند بدء تشغيل العملية ، قم بإنشاء اتصال غير متزامن مع www.baidu.com وإرسال البيانات للحصول على البيانات
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // تعيين اتصال مشفر ssl 
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "اتصال ناجح\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "تم إغلاق الاتصال\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "كود الخطأ: $code، الرسالة: $msg\n";
    };
    $connection_to_baidu->connect();
};

// تشغيل العامل
Worker::runAll();
```
