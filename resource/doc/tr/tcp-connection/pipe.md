# pipe
## Açıklama:
```php
void Connection::pipe(TcpConnection $target_connection)
```

## Parametre:
Mevcut bağlantının veri akışını hedef bağlantıya yönlendirir. Dahili olarak trafiği kontrol eder. Bu yöntem TCP proxy için çok kullanışlıdır.

## Örnek TCP proxy:

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// tcp bağlantısı kurulduktan sonra
$worker->onConnect = function(TcpConnection $connection)
{
    // Yerel 80 portuna asenkron bir bağlantı oluşturulur
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Mevcut istemci bağlantısının verilerini 80 portuna bağlantıya yönlendirme ayarını yapar
    $connection->pipe($connection_to_80);
    // 80 port bağlantısından dönen verileri istemci bağlantısına yönlendirir
    $connection_to_80->pipe($connection);
    // Asenkron bağlantıyı gerçekleştirir
    $connection_to_80->connect();
};

// Worker'ı çalıştırma
Worker::runAll();
```
