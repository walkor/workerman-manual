# workerman/crontab

# Açıklama
`workerman/crontab`, workerman'a dayalı zamanlanmış bir görev programıdır, linux'un crontab'ına benzer. `workerman/crontab`, saniye düzeyinde zamanlama destekler.

>`workerman/crontab` kullanmak için öncelikle PHP'nin zaman dilimini ayarlamanız gerekir, aksi takdirde çalıştırma sonuçları beklenenden farklı olabilir.

## Zaman Açıklaması
```
0   1   2   3   4   5
|   |   |   |   |   |
|   |   |   |   |   +------ haftanın günü (0 - 6) (Pazar=0)
|   |   |   |   +------ ay (1 - 12)
|   |   |   +-------- ayın günü (1 - 31)
|   |   +---------- saat (0 - 23)
|   +------------ dakika (0 - 59)
+-------------- saniye (0-59) [opsiyonel, eğer yoksa, en küçük zaman hassasiyeti dakikadır]
```

# Kurulum
```composer require workerman/crontab```

# Örnek
```php
<?php
use Workerman\Worker;
require __DIR__ . '/vendor/autoload.php';

use Workerman\Crontab\Crontab;
$worker = new Worker();

// Beklenen sonuçla uyumsuzlukları önlemek için zaman dilimini ayarlayın
date_default_timezone_set('PRC');

$worker->onWorkerStart = function () {
    // Her dakikanın 1. saniyesinde çalıştır.
    new Crontab('1 * * * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
    // Günde 7.50'de çalıştır, burada saniye kısmı ihmal edilmiştir.
    new Crontab('50 7 * * *', function(){
        echo date('Y-m-d H:i:s')."\n";
    });
};

Worker::runAll();
```

> Not: Zamanlanmış görevler hemen çalıştırılmaz, tüm zamanlanmış görevler bir sonraki dakikaya girilene kadar hesaplanmaya başlamaz.

# API
**Crontab::destroy()**

Zamanlayıcıyı yok et
```$crontab = new Crontab('1 * * * * *', function(){
    echo date('Y-m-d H:i:s')."\n";
});
$crontab->destroy();```
