```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
نشر حدث معين، سيتلقى جميع المشتركين في هذا الحدث هذا الحدث ويشغلون الوظيفة المسجلة بواسطة `on($event_name, $callback)`.

### المعاملات
 ``` $event_name ```

اسم الحدث الذي سيتم نشره، ويمكن أن يكون أي سلسلة نصية. إذا لم يكن للحدث أي مشتركين، سيتم تجاهل الحدث.

 ``` $event_data ```

البيانات المتعلقة بالحدث، يمكن أن تكون أرقام أو سلاسل نصية أو أرقام. 

### العائد
void

### مثال
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function($http_worker)
{
    Channel\Client::connect('127.0.0.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $event_name = 'user_login';
    $event_data = array('uid'=>123, 'uname'=>'tom');
    Channel\Client::publish($event_name, $event_data );
};

Worker::runAll();
```
