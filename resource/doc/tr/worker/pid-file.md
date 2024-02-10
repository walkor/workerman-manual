# pidFile
## Açıklama:
```php
static string Worker::$pidFile
```

Özel bir gereksinim yoksa, bu özelliğin belirtilmemesi önerilir.

Bu özellik, WorkerMan sürecinin pid dosya yolunu belirlemek için kullanılan global bir statik özelliktir.

Bu ayar izlemede oldukça faydalıdır, örneğin WorkerMan'in pid dosyasının sabit bir dizine yerleştirilmesi, bazı izleme yazılımlarının pid dosyasını okuyarak WorkerMan süreç durumunu izlemesini kolaylaştırır.

Eğer belirtilmezse, WorkerMan varsayılan olarak Workerman dizininin yanında (workerman3.2.3'ten önceki sürümlerde ```sys_get_temp_dir()``` altında) pid dosyasını otomatik olarak oluşturur ve birden çok WorkerMan örneğinin başlatılmasından kaynaklanan pid çakışmasını önlemek için WorkerMan, mevcut WorkerMan'in yolunu içeren pid dosyası oluşturur.

Not: Bu özellik, etkin olması için ```Worker::runAll();``` çalıştırılmadan önce ayarlanmalıdır. Windows sistemler bu özelliği desteklemez.


## Örnek

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Worker çalıştır
Worker::runAll();
```
