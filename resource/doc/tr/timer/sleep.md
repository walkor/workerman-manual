# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
`Timer::sleep()` işlevi, PHP'nin yerleşik `sleep()` işlevine benzer, fakat farkı `Timer::sleep()` işlevinin blokaj oluşturmamasıdır (mevcut işlemi bloke etmez).

> **Not**
> Bu özellik workerman>=5.0 gerektirir.
> Bu özellik için composer require revolt/event-loop ^1.0.0 kurulumu gereklidir, ya da olay tabanlı olarak Swoole/Swow kullanılmalıdır.

### Parametre
``` delay ```

Kaç saniyede bir çalıştırılacağı. Saniye cinsinden ifade edilir, ondalık sayılarla ifade edilebilir, en küçük değeri 0.001, yani milisaniye cinsinden ifade edilebilir.

### Dönüş Değeri
Dönüş değeri bulunmamaktadır.

### Örnek

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // 1.5 saniye gecikmeli olarak veri gönder
    Timer::sleep(1.5);
    // Veri gönder
    $connection->send('merhaba workerman');
};

Worker::runAll();
```
