# maxSendBufferSize
## Açıklama:
```php
int Connection::$maxSendBufferSize
```

Her bağlantının ayrı bir uygulama katmanı gönderi önbelleği vardır. Eğer istemci tarafının alım hızı, sunucu tarafının gönderme hızından düşükse, veri uygulama katmanı önbelleğinde gönderilmek üzere bekletilir.

Bu özellik, mevcut bağlantının uygulama katmanı gönderi önbellek boyutunu ayarlamak için kullanılır. Varsayılan olarak ayarlanmamışsa [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md)'a göre (1 MB) ayarlanır.

Bu özellik, [onBufferFull](../worker/on-buffer-full.md) geri çağrısını etkiler.


## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Mevcut bağlantının uygulama katmanı gönderi önbellek boyutunu 102400 bayt olarak ayarla
    $connection->maxSendBufferSize = 102400;
};
// Worker'ı çalıştır
Worker::runAll();
```
