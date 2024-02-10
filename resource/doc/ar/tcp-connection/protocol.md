# البروتوكول

## الوصف:
```php
string Connection::$protocol
```

تعيين فصيل الاتصال الحالي


## مثال

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // عند الإرسال، سيتم استدعاء$connection->protocol::encode() تلقائيًا، لتعبئة البيانات ثم الإرسال
    $connection->send("hello");
};
// تشغيل الخادم
Worker::runAll();
```
