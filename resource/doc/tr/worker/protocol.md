# protokol
Gereksinimler ```(workerman >= 3.2.7)```

## Açıklama:
```php
string Worker::$protocol
```

Mevcut Worker örneğinin protokol sınıfını ayarlar.

Not: Protokol işleme sınıfı, Worker'ı dinleme parametreleri başlatılırken doğrudan belirtilebilir. Örneğin
```php
$worker = new Worker('http://0.0.0.0:8686');
```


## Örnek

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Worker'ı çalıştır
Worker::runAll();
```


Yukarıdaki kod aşağıdaki kodla eşdeğerdir

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * İlk olarak, kullanıcının özel \Protocols\Http protokol sınıfı olup olmadığına bakılır,
 * Eğer yoksa workerman dahili protokol sınıfı Workerman\Protocols\Http kullanılır
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Worker'ı çalıştır
Worker::runAll();
```
