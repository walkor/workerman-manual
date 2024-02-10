# onWorkerReload
Gereksinimler (workerman >= 3.2.5)

## Açıklama:
```php
callback Worker::$onWorkerReload
```
Bu özellik genellikle kullanılmaz.

Worker'a reload sinyali aldığında çalıştırılacak geri çağırımı ayarlar.

onWorkerReload geri çağırımını kullanarak, iş yapabilirsiniz, örneğin, işlemin yeniden başlatılmasını gerektirmeyen durumlarda iş yapılandırma dosyalarını yeniden yükleyebilirsiniz.

**Not**:

Alt işlem, reload sinyali aldıktan sonra varsayılan olarak çıkış yaparak yeniden başlatılır, böylece yeni işlem, kod güncellemesini tamamlamak için iş mantığını yeniden yükler. Bu nedenle reload sonrası alt işlemin onWorkerReload geri çağırımını tamamladıktan hemen sonra çıkış yapması normal bir durumdur.

Reload sinyali alındıktan sonra alt işlemin sadece onWorkerReload'u çalıştırmasını ve çıkış yapmasını istemiyorsanız, Worker örneğinin reloadable özelliğini false olarak ayarlayabilirsiniz.

## Geri çağırım fonksiyonunun parametreleri
``` $worker ```
Yani Worker nesnesi

## Örnek
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// reloadable false olarak ayarlandı, yani alt işlem reload sinyali alırsa yeniden başlatılmaz
$worker->reloadable = false;
// Reload sonrası tüm istemcilere sunucunun yeniden başlatıldığını bildirir
$worker->onWorkerReload = function(Worker $worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send('worker reloading');
    }
};
// Worker çalıştırılır
Worker::runAll();
```
Not: Geri çağırım olarak anonim fonksiyon kullanmanın yanı sıra, [buraya bakarak](../faq/callback_methods.md) diğer geri çağırım yazım türlerini de kullanabilirsiniz.
