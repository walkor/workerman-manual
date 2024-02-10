# ulaştırma
## Açıklama:
```php
string Worker::$transport
```

Mevcut Worker örneğinin kullandığı taşıma katmanı protokolünü ayarlar. Şu anda sadece 3 türü destekler (tcp, udp, ssl). Ayarlanmadığında varsayılan tcp olarak belirlenir.

``` Not: ssl, Workerman sürümü>=3.3.7 gerektirir. ```


## Örnek 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// udp protokolünü kullan
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// worker'ı çalıştır
Worker::runAll();
```


## Örnek 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Sertifika en iyi şekilde alınmış olmalıdır
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // ayrıca crt dosyası da olabilir
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Burada websocket protokolü ayarlanmıştır
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// transport'u ssl olarak ayarla, websocket+ssl yani wss'yi temsil eder
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
