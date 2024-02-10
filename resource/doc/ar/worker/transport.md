# النقل
## شرح:
```php
string Worker::$transport
```

يُعين بروتوكول النقل الذي يستخدمه العامل الحالي، حيث تدعم الآن ثلاثة أنواع فقط (tcp، udp، ssl). إذا لم يتم تعيينه سيكون الافتراضي tcp.

``` ملاحظة: يتطلب SSL إصدار Workerman> = 3.3.7 ```

## مثال 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// استخدام بروتوكول UDP
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// تشغيل worker
Worker::runAll();
```

## مثال 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// يفضل أن يكون الشهادة شهادة مصدقة
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // يمكن أيضًا أن يكون ملف crt
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// في هذا المثال نحدد بروتوكول websocket
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// تعيين النقل لتفعيل SSL، websocket + ssl أي wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
