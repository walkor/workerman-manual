# Kullanıcı

## Açıklama:
```php
string Worker::$user
```

Mevcut Worker örneğinin hangi kullanıcı tarafından çalıştırılacağını ayarlar. Bu özellik yalnızca mevcut kullanıcı root ise etkilidir. Ayarlanmadığında varsayılan olarak mevcut kullanıcı tarafından çalıştırılır.

```$user```'ın düşük izinli bir kullanıcı olarak ayarlanması önerilir, örneğin www-data, apache, nobody vb.

Not: Bu özellik sadece ```Worker::runAll();``` çalıştırılmadan önce ayarlanabilir. Windows sistemlerinde bu özellik desteklenmemektedir.

## Örnek

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Örneğin çalıştırılacak kullanıcıyı ayarla
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Worker başlıyor...\n";
};
// Worker'ı çalıştır
Worker::runAll();
```
