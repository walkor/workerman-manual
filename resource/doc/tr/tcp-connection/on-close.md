# onClose
## Açıklama:
```php
Geribildirim Connection::$onClose
```

Bu geri çağrı [Worker::$onClose](../worker/on-close.md) geri çağrısıyla aynı işlevi görür, farkı sadece geçerli bağlantı için geçerlidir, yani belirli bir bağlantı için onClose geri çağrısını ayarlayabilirsiniz.

## Örnekler

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Bağlantı olayı gerçekleştiğinde tetiklenir
$worker->onConnect = function(TcpConnection $connection)
{
    // Bağlantının onClose geri çağrısını ayarlayın
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "bağlantı kapatıldı\n";
    };
};
// Worker'ı çalıştır
Worker::runAll();
```

Yukarıdaki kod aşağıdaki kodla aynı etkiye sahiptir

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Tüm bağlantıların onclose geri çağrısını ayarlayın
$worker->onClose = function(TcpConnection $connection)
{
    echo "bağlantı kapatıldı\n";
};
// Worker'ı çalıştır
Worker::runAll();
```
