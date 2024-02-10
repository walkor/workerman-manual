# kimlik

## Açıklama:
```php
int Connection::$id
```

Bağlantının kimliği. Bu artan bir tamsayıdır.

Not: Workerman çoklu işlemli bir yapıya sahiptir. Her işlem kendi içinde artan bir bağlantı kimliği tutar, bu nedenle farklı işlemler arasında bağlantı kimlikleri tekrarlanabilir. Tekrarlanmayan bir bağlantı kimliği isteniyorsa, connection->id'ye istenen şekilde yeniden bir değer atamak gereklidir, örneğin worker->id ön ekini eklemek.

## Ayrıca bakınız
[Worker'ın connections özelliği](../worker/connections.md)

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Worker'ı başlat
Worker::runAll();
```
