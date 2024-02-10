# resumeRecv
## Açıklama:
```php
void Connection::resumeRecv(void)
```

Mevcut bağlantının veri almaya devam etmesini sağlar. Bu yöntem, Connection::pauseRecv ile kullanılarak yük akışı kontrolü için çok uygundur.

## Parametreler

Parametre yok

## Örnek

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Bağlantı nesnesine dinamik olarak eklenecek bir özellik, şu anda kaç isteğin gönderildiğini saklamak için kullanılır
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Her bağlantı 100 istek alırsa daha fazla veri almayı durdurur
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // 30 saniye sonra veri almayı tekrar başlat
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Worker çalıştırma
Worker::runAll();
```

## Ayrıca bakınız
void Connection::pauseRecv(void) Bağlantı nesnesinin veri almaya durmasını sağlar
