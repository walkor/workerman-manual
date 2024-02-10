# logFile
## Açıklama:
```php
static string Worker::$logFile
```

workerman günlük dosyasının konumunu belirtmek için kullanılır.

Bu dosya, workerman'ın kendi ilgili günlüklerini kaydeder, başlatma, durdurma vb. işlemleri içerir.

Eğer belirtilmemişse, dosya adı varsayılan olarak workerman.log olur ve konumu Workerman'ın bir üst dizinindedir.

**Not:**

Bu günlük dosyası yalnızca workerman'ın kendi başlatma ve durdurma gibi ilgili günlükleri kaydeder, herhangi bir iş günlüğünü içermez.

İş günlüğü sınıfı, [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) veya [error_log](https://php.net/manual/zh/function.error-log.php) gibi işlevleri kullanarak kendisi uygulayabilir.

## Örnek

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// worker'ı başlat
Worker::runAll();
```
