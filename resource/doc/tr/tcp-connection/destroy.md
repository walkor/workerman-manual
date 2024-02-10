# destroy
## Açıklama:
```php
void Connection::destroy()
```

Bağlantıyı hemen kapatır.

Kapatma işleminden farklı olarak, destroy çağrıldığında bağlantının gönderim önbelleğinde henüz gönderilmemiş veri olsa bile bağlantı hemen kapatılır ve bağlantının```onClose``` geri çağrımı hemen tetiklenir.

## Parametre

Parametre yok

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // if something wrong
    $connection->destroy();
};
// Worker'ı başlat
Worker::runAll();
```
