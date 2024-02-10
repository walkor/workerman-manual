# worker
## Açıklama:
```php
Worker Connection::$worker
```

Bu özellik salt okunur bir özelliktir, yani mevcut bağlantı nesnesinin ait olduğu işçi örneğidir.


## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Bir istemci veri gönderdiğinde, mevcut işlem tarafından yönetilen diğer tüm istemcilere iletim yapar
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// İşçiyi çalıştır
Worker::runAll();
```
