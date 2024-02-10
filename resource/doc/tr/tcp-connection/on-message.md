# onMessage
## Açıklama:
```php
geriçağırma Bağlantı::$onMessage
```

[Worker::$onMessage](../worker/on-message.md) geriçağırımıyla aynı işlevi görür, farkı yalnızca geçerli bağlantı için geçerlidir, yani belirli bir bağlantı için onMessage geriçağırımı ayarlanabilir.

## Örnekler

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// İstemci bağlantısı olduğunda
$worker->onConnect = function(TcpConnection $connection)
{
    // Bağlantının onMessage geri çağırımını ayarla
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('alım başarılı');
    };
};
// Worker'ı çalıştır
Worker::runAll();
```

Yukarıdaki kod, aşağıdaki kodla aynı etkiye sahiptir

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Tüm bağlantıların onMessage geri çağırımını doğrudan ayarla
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('alım başarılı');
};
// Worker'ı çalıştır
Worker::runAll();
```
