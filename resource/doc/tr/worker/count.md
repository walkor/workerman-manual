# count

## Açıklama:
```php
int Worker::$count
```

Bu özellik, mevcut Worker örneğinin kaç tane işlem başlatacağını belirler. Belirtilmediği takdirde varsayılan değer 1'dir.

İşlem sayısının nasıl belirleneceği için [**buraya**](../faq/processes-count.md) bakınız.

Not: Bu özellik sadece ```Worker::runAll();``` çalıştırılmadan önce ayarlanmalıdır. Windows işletim sistemini bu özellik desteklemez.

## Örnek

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 8484 portunda WebSocket protokolü ile hizmet sunarken 8 işlem başlat
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "İşlem başlatılıyor...\n";
};
// İşçiyi çalıştır
Worker::runAll();
```
