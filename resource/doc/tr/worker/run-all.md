# runAll
```php
void Worker::runAll(void)
```
Tüm Worker örneklerini çalıştırır.

**Dikkat:**

Worker::runAll() çalıştırıldıktan sonra sürekli olarak engellenecektir, yani Worker::runAll() fonksiyonundan sonraki kodlar çalıştırılmayacaktır. Tüm Worker örneklemeleri Worker::runAll() fonksiyonundan önce gerçekleştirilmelidir.

### Parametre
Parametre yok

### Dönüş Değeri
Dönüş yok

## Örnek: Birden fazla Worker örneğini çalıştırma

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Tüm Worker örneklerini çalıştırma
Worker::runAll();
```


**Not:**

Windows versiyonu Workerman'da aynı dosyada birden fazla Worker örneği oluşturulamaz.
Yukarıdaki örnek, Windows versiyonu Workerman'da çalıştırılamaz.

Windows versiyonu Workerman'da birden fazla Worker örneği oluşturmak için aşağıdaki gibi farklı dosyalara yerleştirilmelidir:

start_http.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello http');
};

// Tüm Worker örneklerini çalıştırma (sadece bir örnek burada)
Worker::runAll();
```

start_websocket.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:4567');
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello websocket');
};

// Tüm Worker örneklerini çalıştırma (sadece bir örnek burada)
Worker::runAll();
```
