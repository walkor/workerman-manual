# globalEvent

## Açıklama:
```php
static Event Worker::$globalEvent
```

Bu özellik global bir statik özelliktir. Global eventloop örneğidir ve dosya tanımlayıcılarının okuma yazma olaylarına veya sinyal olaylarına kaydedilebilir.

## Örnek

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // İşlem SIGALRM sinyalini aldığında bazı bilgileri yazdır
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Sinyal SIGALRM alındı\n";
    });
};
// Worker'ı çalıştır
Worker::runAll();
```

## Test
Workerman başladıktan sonra geçerli işlem kimlik numarasını (bir sayı) döndürür. Komut istemine aşağıdaki komutu girin
```   
kill -SIGALRM işlemkimliği
```
Sunucu aşağıdakini yazdıracaktır
```
Sinyal SIGALRM alındı
```
