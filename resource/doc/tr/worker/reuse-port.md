# reusePort
> **Not** 
> Requires workerman >= 3.2.1, PHP >= 7.0, Windows and Mac OS do not support this feature

## Explanation:

```php
bool Worker::$reusePort
```

Belirtilen çalışanın aynı TCP portunu paylaşarak birden fazla işlemci sürecinin istemci bağlantılarını kabul etmesine izin verilip verilmeyeceğini ayarlar. 

TCP portu yeniden kullanımı (socket'in SO_REUSEPORT seçeneği ile) olduğunda, akrabalık olmayan ve birbirleriyle ilgisi olmayan birden fazla işlemci süreci aynı portu dinleyebilir ve sistem çekirdeği tarafından yük dengelemesi yapılır, hangi sürecin soket bağlantısını hangi işlemcinin işleyeceğine karar vererek sürpriz etkisini önler, çoklu işlemci kısa bağlantı uygulamalarının performansını artırabilir.

**Not:** Bu özellik için PHP sürümü >=7.0 olmalıdır.

## Örnek 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Worker'ı çalıştır
Worker::runAll();
```

## Örnek 2: workerman tarafından çoklu protokol dinleme
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Her bir süreç başladığında, mevcut süreçte yeni bir dinleme eklenir
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Birden fazla süreç aynı portu dinlerse (dinlenen soket ebeveyn süreçten devralınmazsa)
     * Port yeniden kullanımının açık olması gerekir, aksi takdirde "Address already in use" hatası alınır.
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Dinlemeyi başlat
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Worker'ı çalıştır
Worker::runAll();
```
