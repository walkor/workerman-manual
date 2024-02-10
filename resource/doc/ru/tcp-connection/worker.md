```php
# worker
## Описание:
```php
Worker Connection::$worker
```

Это свойство только для чтения и представляет собой экземпляр worker, к которому относится текущий объект connection


## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');

// Когда клиент отправляет данные, пересылать их всем остальным клиентам, поддерживаемым текущим процессом
$worker->onMessage = function(TcpConnection $connection, $data)
{
    foreach($connection->worker->connections as $con)
    {
        $con->send($data);
    }
};
// Запуск worker
Worker::runAll();
```
