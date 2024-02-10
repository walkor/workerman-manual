# getRemotePort
## Açıklama:
```php
int Connection::getRemotePort()
```

Bu bağlantının istemci bağlantı noktasını alır.

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
    echo "Adresten yeni bağlantı " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Worker'ı çalıştır
Worker::runAll();
```
