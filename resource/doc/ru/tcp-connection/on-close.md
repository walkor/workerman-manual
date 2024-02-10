# onClose
## Описание:
```php
callback Connection::$onClose
```

Этот обратный вызов имеет тот же эффект, что и обратный вызов [Worker::$onClose](../worker/on-close.md), за исключением того, что он действует только для текущего соединения, то есть можно установить обратный вызов onClose для определенного соединения.

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Когда происходит событие подключения
$worker->onConnect = function(TcpConnection $connection)
{
    // Установить обратный вызов onClose для соединения
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "соединение закрыто\n";
    };
};
// Запустить worker
Worker::runAll();
```

Приведенный выше код аналогичен следующему:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Установить обратный вызов onClose для всех соединений
$worker->onClose = function(TcpConnection $connection)
{
    echo "соединение закрыто\n";
};
// Запустить worker
Worker::runAll();
```
