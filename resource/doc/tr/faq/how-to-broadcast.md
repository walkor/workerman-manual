# Veri Yayını (Yayın) Nasıl Yapılır

## Örnek (Zamanlanmış Veri Yayını)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Bu örnekte, işlem sayısı 1 olmalı
$worker->count = 1;
// İşlem başlatıldığında tüm istemci bağlantılarına düzenli aralıklarla veri göndermek için bir zamanlayıcı ayarlayın
$worker->onWorkerStart = function($worker)
{
    // Her 10 saniyede bir düzenli olarak veri gönder
    Timer::add(10, function() use ($worker)
    {
        // Mevcut işlemdeki tüm istemci bağlantıları üzerinde dolaşarak mevcut sunucu zamanını gönder
        foreach ($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// İşlemi çalıştır
Worker::runAll();
```

## Örnek (Grup Sohbeti)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Bu örnekte, işlem sayısı 1 olmalı
$worker->count = 1;
// İstemci bir mesaj gönderdiğinde, diğer kullanıcılara yayın yapın
$worker->onMessage = function(TcpConnection $connection, $message) use ($worker)
{
    foreach ($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// İşlemi çalıştır
Worker::runAll();
```

## Açıklama:
**Tek İşlem:**
Yukarıdaki örnekler sadece **tek işlem** (```$worker->count=1```) kullanabilir çünkü birden fazla işlemde birden fazla istemcinin farklı işlemlere bağlanabileceği, işlemler arasındaki istemcilerin izole olduğu ve doğrudan iletişim kuramadığı bir durum mevcuttur, yani A işleminde B işleminin istemci bağlantısını doğrudan manipüle edemez ve veri gönderemez. (Bu durumu gerçekleştirmek için işlemler arası iletişim gereklidir, örneğin, Kanal bileşenini kullanabilirsiniz, örneğin [Cluster Send Example](../components/channel-examples.md), [Group Send Example](../components/channel-examples2.md)).

**GatewayWorker Tavsiyesi:**
Workerman üzerine geliştirilmiş GatewayWoker çerçevesi, daha kolay ve kullanışlı bir itme mekanizması sağlar, multicast, broadcast vb. ayarlayabilir, çoklu işlem veya çoklu sunucu dağıtımı için yapılandırılabilir. İstemcilere veri göndermek için GatewayWorker çerçevesini kullanmanız önerilir.

GatewayWorker el kitabı adresi: https://doc2.workerman.net
