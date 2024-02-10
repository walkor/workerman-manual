# bağlantılar
## Açıklama:
```php
array Worker::$connections
```

Formatı şu şekildedir:
```php
array(id=>bağlantı, id=>bağlantı, ...)
```

Bu özellik, **mevcut süreç** içindeki tüm istemci bağlantı nesnelerini depolar. Burada id, bağlantının id numarasıdır, detaylar için [TcpConnection id özelliğine](../tcp-connection/id.md) bakın.

```$connections```, yayın yaparken veya bağlantı id'sine göre bağlantı nesnesi elde etmek için oldukça yararlıdır.

Eğer bağlantı numarasının ```$id``` olduğunu biliyorsanız, ilgili bağlantı nesnesini kolayca ```$worker->connections[$id]``` kullanarak elde edebilir ve bu şekilde ilgili soket bağlantısını işleyebilirsiniz, örneğin ```$worker->connections[$id]->send('...')``` şeklinde veri gönderebilirsiniz.

Not: Bağlantı kapatıldığında (onClose tetiklenirse), ilgili ```bağlantı``` ```$connections``` dizisinden kaldırılacaktır.

Not: Geliştiricilerin bu özelliği değişiklik yapmaması gerekmektedir, aksi takdirde öngörülemeyen durumlara neden olabilir.

## Örnek

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// İşçi başlatıldığında tüm istemci bağlantılarına düzenli aralıklarla veri göndermek için bir zamanlayıcı ayarlayın
$worker->onWorkerStart = function($worker)
{
    // Her 10 saniyede bir zamanlayıcı
    Timer::add(10, function() use ($worker)
    {
        // Mevcut süreçteki tüm istemci bağlantılarını dolaşarak mevcut sunucu zamanını gönderin
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// İşçiyi çalıştırın
Worker::runAll();
```
