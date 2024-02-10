# stdoutFile
## Açıklama:
```php
static string Worker::$stdoutFile
```

Bu özellik, global bir statik özelliktir. Eğer daemon modunda (`-d` ile başlatıldığında), tüm terminal çıktıları (echo var_dump vb.) stdoutFile belirtilen dosyaya yönlendirilecektir.

Eğer belirtilmezse ve daemon modunda çalıştırılıyorsa, tüm terminal çıktıları varsayılan olarak `/dev/null` dosyasına yönlendirilir (yani tüm çıktılar varsayılan olarak yok edilir).

> Not: `/dev/null`, Linux'te özel bir dosyadır ve aslında bir siyahtır. Bu dosyaya yazılan tüm veriler atılır. Çıktıları atmak istemiyorsanız, `Worker::$stdoutFile`'ı normal bir dosya yoluna ayarlayabilirsiniz.

> Not: Bu özellik, `Worker::runAll();` çalıştırılmadan önce ayarlanmalıdır. Windows sistemi bu özelliği desteklemez.

## Örnek

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Tüm print çıktıları /tmp/stdout.log dosyasında saklanacaktır
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker başlatıldı\n";
};
// Worker çalıştırılıyor
Worker::runAll();
```
