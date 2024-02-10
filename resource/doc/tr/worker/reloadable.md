# reloadable
## Açıklama:
```php
bool Worker::$reloadable
```

`php start.php reload` komutu çalıştırıldığında tüm alt işlemlere yeniden yükleme sinyali (SIGUSR1) gönderilir.

Alt işlem yeniden yükleme sinyali aldığında otomatik olarak kapanır ve ana işlem otomatik olarak yeni bir işlem başlatır, genellikle iş mantığını güncellemek için kullanılır.

$reloadable özelliği false olduğunda, reload sinyali alındığında yalnızca [onWorkerReload](on-worker-reload.md) olayını tetikler ve mevcut işlemi yeniden başlatmaz.

Örneğin Gateway/Worker modelinde, gateway işlemi müşteri bağlantılarını sürdürürken, worker işlemi istekleri işler.
Gateway işleminin reloadable özelliği false olarak ayarlanırsa, müşteri bağlantılarını kapatmadan iş mantığını güncelleyebilirsiniz.


## Örnek
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Bu örneğin reload sinyali aldığında yeniden başlatılıp başlatılmayacağını ayarlayın
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "İşlem başlatılıyor...\n";
};
// Tüm işlemleri çalıştır
Worker::runAll();
```
