# daemonize
## Açıklama:
```php
static bool Worker::$daemonize
```

Bu özellik, global olarak statik olarak tanımlanmış olup, daemon (bekçi süreci) şeklinde mi çalışılıp çalışılmayacağını belirtir. Başlatma komutu ```-d``` parametresini kullanıyorsa, bu özellik otomatik olarak true olarak ayarlanır. Ayrıca kod içinde manuel olarak da belirlenebilir.

Not: Bu özellik, etkili olabilmesi için ```Worker::runAll();``` çalıştırılmadan önce ayarlanmalıdır. Windows sistemlerinde bu özellik desteklenmez.

## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Çalışan başladı\n";
};
// Worker'ı çalıştır
Worker::runAll();
```
