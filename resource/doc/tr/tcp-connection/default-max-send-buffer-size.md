# defaultMaxSendBufferSize
## Açıklama:
```php
static int Connection::$defaultMaxSendBufferSize
```

Bu özellik, tüm bağlantıların varsayılan uygulama katmanı gönderme önbellek boyutunu ayarlamak için kullanılan global bir statik özelliktir. Varsayılan olarak ayarlanmadıysa, ```1MB``` olarak ayarlanır. ```Connection::$defaultMaxSendBufferSize``` dinamik olarak ayarlanabilir, ayarlandıktan sonra sadece sonrasında oluşturulan yeni bağlantılar için geçerlidir.

Bu özellik, [onBufferFull](../worker/on-buffer-full.md) geri çağırımını etkiler.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Tüm bağlantıların varsayılan uygulama katmanı gönderme önbellek boyutunu ayarlayın
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Geçerli bağlantının uygulama katmanı gönderme önbellek boyutunu ayarlayın, varsayılan değeri geçersiz kılacaktır
    $connection->maxSendBufferSize = 4*1024*1024;
};
// Worker'ı çalıştırın
Worker::runAll();
```
