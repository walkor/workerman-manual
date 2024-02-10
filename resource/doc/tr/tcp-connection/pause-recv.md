# pauseRecv
## Açıklama:
```php
void Connection::pauseRecv(void)
```

Mevcut bağlantının veri alımını durdurur. Bağlantının onMessage geri çağrısı tetiklenmeyecektir. Bu yöntem veri akışı kontrolü için çok kullanışlıdır.

## Parametreler

Parametre yok

## Örnekler

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Bağlantı nesnesine dinamik olarak ek bir özellik ekleyerek, mevcut bağlantının kaç istek gönderdiğini saklar
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Her bağlantı 100 istek aldıktan sonra veri alımını durdur
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Worker'ı çalıştır
Worker::runAll();
```

## Bakınız
void Connection::resumeRecv(void) Karşılık gelen bağlantı nesnesinin veri alımını yeniden başlatır
