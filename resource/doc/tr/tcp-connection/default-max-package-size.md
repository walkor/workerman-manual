# maxPackageSize

## Açıklama:
```php
static int Connection::$defaultMaxPackageSize
```

Bu özellik, her bağlantının alabileceği maksimum paket uzunluğunu ayarlamak için kullanılan global bir statik özelliktir. Varsayılan olarak ayarlanmazsa, 10MB olarak kabul edilir.

Gelen veri paketinin ayrıştırılması (protokol sınıfının input yöntemi dönüş değeri) ```Connection::$defaultMaxPackageSize```'den büyük ise, bu veri yasadışı olarak kabul edilir ve bağlantı kesilir.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Her bağlantının alabileceği veri paketinin maksimum uzunluğunu 1024000 bayt olarak ayarlayın
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Worker'ı çalıştırın
Worker::runAll();
```
