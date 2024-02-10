# getRemoteIp
## Описание:
```php
string Connection::getRemoteIp()
```

Получение IP-адреса клиента для данного соединения.

## Параметры

Без параметров

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "новое соединение с IP " . $connection->getRemoteIp() . "\n";
};
// Запуск воркера
Worker::runAll();
```
