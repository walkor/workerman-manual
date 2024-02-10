# İsim

## Açıklama:
```php
string Worker::$name
```

Mevcut Worker örneğinin adını ayarlar, böylece süreci tanımak için status komutunu çalıştırdığınızda kullanabilirsiniz. Ayarlanmadığında varsayılan olarak none olarak ayarlanır.

## Örnek

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Örneğin adını ayarla
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};
// Worker'ı çalıştır
Worker::runAll();
```
