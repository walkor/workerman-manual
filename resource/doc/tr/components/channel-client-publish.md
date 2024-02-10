```php
# yayınla (Workerman sürümü >=3.3.0 gereklidir)

void \Channel\Client::publish(string $event_name, mixed $event_data)
```
Bir olayı yayınlar, bu olayın aboneleri olayı alır ve ```on($event_name, $callback)``` ile kayıtlı olan ```$callback``` gerçekleştirilir.

### Parametreler
 ``` $event_name ```

Yayınlanacak olayın adı, herhangi bir dize olabilir. Eğer olayın herhangi bir abonesi yoksa, olay görmezden gelinir.

 ``` $event_data ```

Olayla ilgili veri, sayı, dize veya dizi olabilir.

### Dönüş Değeri
void

### Örnek
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
