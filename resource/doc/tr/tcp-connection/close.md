# Kapat
## Açıklama:
```php
void Connection::close(mixed $data = '')
```

Güvenli bir şekilde bağlantıyı kapatır.

close çağrısı, bağlantının kapatılması için gönderi tamponundaki verilerin gönderilmesini bekler ve ardından bağlantının `onClose` geri çağırımını tetikler.

## Parametreler

 ``` $data ```

Opsiyonel parametre, gönderilmek istenen veri (bir protokol belirtilmişse, `encode` yöntemini kullanarak `data` verisini paketler). Veri gönderildikten sonra bağlantı kapatılacak ve ardından `onClose` geri çağırımı tetiklenir.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// Worker'ı çalıştır
Worker::runAll();
```
