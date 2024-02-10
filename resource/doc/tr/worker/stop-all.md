# stopAll
```php
void Worker::stopAll(void)
```

Mevcut işlemi durdurur ve çıkar. 

> **Not**
> `Worker::stopAll()`, mevcut işlemi durdurmak için kullanılır. Mevcut işlem çıktıktan sonra ana işlem hemen yeni bir işlem başlatır. Eğer tüm workerman servisini durdurmak istiyorsanız, `posix_kill(posix_getppid(), SIGINT)` kullanmalısınız.

### Parametre
Parametre yok

### Dönüş Değeri
Dönüş yok

## Örnek: max_request

Aşağıdaki örnek, alt işlem her 1000 isteği işledikten sonra stopAll'ı çalıştırarak çıkış yapar ve yeni bir süreci yeniden başlatır. Bu, php-fpm'nin max_request özelliğine benzer, temel olarak php iş kodlarının neden olduğu bellek sızıntısı sorunlarını çözmek için kullanılır.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Her işlem en fazla 1000 isteği işler
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Zaten işlenen istek sayısı
    static $request_count = 0;

    $connection->send('hello http');
    // Eğer istek sayısı 1000'e ulaşırsa
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Mevcut işlemi durdurur, ana işlem hemen yeni bir işlem başlatır
         * ve bu şekilde işlem yeniden başlatılır
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
