# getRemotePort
## Описание:
```php
int Connection::getRemotePort()
```

Получение порта клиентского соединения

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
    echo "Новое соединение с адреса " .
    $connection->getRemoteIp() . ":". $connection->getRemotePort() ."\n";
};
// Запуск worker
Worker::runAll();
```
