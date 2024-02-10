```php
void \Channel\Client::publish(string $event_name, mixed $event_data)
```
Veröffentlicht ein bestimmtes Ereignis, das von allen Abonnenten dieses Ereignisses empfangen wird und das das mit ```on($event_name, $callback)``` registrierte Callback```$callback``` auslöst

### Parameter
 ``` $event_name ```

Der Name des veröffentlichten Ereignisses, der beliebig sein kann. Wenn das Ereignis keine Abonnenten hat, wird es ignoriert.

 ``` $event_data ```

Die mit dem Ereignis verbundenen Daten können eine Zahl, einen String oder ein Array sein.

### Rückgabewert
void



### Beispiel
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
