# getRemoteIp
## Açıklama:
```php
string Connection::getRemoteIp()
```

Bu bağlantının istemci IP'sini alır.

## Parametreler

Parametre yok

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "ip adresinden yeni bağlantı " . $connection->getRemoteIp() . "\n";
};
// Worker'ı çalıştır
Worker::runAll();
```
