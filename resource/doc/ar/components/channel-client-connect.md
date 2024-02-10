```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('127.0.0.1', 2206);
};

Worker::runAll();
```
```php
void \Channel\Client::connect([string $listen_ip = '127.0.0.1', int $listen_port = 2206])
```
**``` (متطلبات Workerman> = 3.3.0) ```**
```php
```  قم بالاتصال بقناة / السيرفر
### الباراميترات
``` listen_ip ```
عنوان IP الذي يستمع إليه القناة / السيرفر ، إذا لم يتم تمريره فإن الافتراضي هو ```127.0.0.1```
``` listen_port ```
المنفذ الذي يستمع إليه القناة / السيرفر ، إذا لم يتم تمريره فإن الافتراضي هو 2206
### القيمة المصرجة
void
```
