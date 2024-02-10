```php
boolean \Workerman\Timer::del(int $timer_id)
```
Bir zamanlayıcıyı siler

### Parametre
```timer_id```

Zamanlayıcının id'si, yani add arabiriminden dönen tamsayı

### Dönüş Değeri
boolean

### Örnek
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Kaç adet işlemi zamanlanmış görevi çalıştırmak için başlatılacak, çoklu işlem eş zamanlılık sorunu
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Her 2 saniyede bir çalıştır
    $timer_id = Timer::add(2, function()
    {
        echo "görev çalışıyor\n";
    });
    // 20 saniye sonra tek seferlik bir görevi çalıştır, 2 saniyede bir olan zamanlanmış görevi sil
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// İşçiyi çalıştır
Worker::runAll();
```

### Örnek(Zamanlayıcı geri çağrıda mevcut zamanlayıcıyı silme)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Lütfen geri aramada mevcut zamanlayıcı kimliğini kullanırken referans(&) yöntemiyle içe aktarılması gerekir
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // 10 kez çalıştırdıktan sonra zamanlayıcıyı sil
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// İşçiyi çalıştır
Worker::runAll();
```
