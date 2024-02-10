# onWorkerStart
## Açıklama:
```php
geriçağırma Worker::$onWorkerStart
```

Worker alt işlemi başladığında çalışacak geriçağırma işlevini ayarlar, her alt işlem başladığında çalıştırılır.

Not: onWorkerStart alt işlem başlatıldığında çalışır, eğer birden fazla alt işlem başlatılmışsa (```$worker->count > 1```) her alt işlem bir kez çalıştırılır ve toplamda ```$worker->count``` kez çalıştırılır.


## Geriçağırma işlevinin parametreleri

 ``` $worker ```

Yani Worker nesnesi


## Örnek


```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "İşçi başlıyor...\n";
};
// İşçiyi çalıştır
Worker::runAll();
```

Not: Anonim işlev olarak geriçağırma kullanmanın dışında, diğer geriçağırma yazım yöntemleri için [buraya bakabilirsiniz](../faq/callback_methods.md).
