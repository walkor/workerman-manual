# Protokol

## Açıklama:
```php
string Connection::$protocol
```

Mevcut bağlantının protokol sınıfını ayarlar.


## Örnek


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    $connection->protocol = 'Workerman\\Protocols\\Http';
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    // send 时会自动调用$connection->protocol::encode()，打包数据后再发送
    $connection->send("hello");
};
// worker'ı çalıştır
Worker::runAll();
```
