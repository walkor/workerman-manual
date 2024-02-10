# onMessage
## Описание:
```php
callback Connection::$onMessage
```

Действие аналогично обратному вызову [Worker::$onMessage](../worker/on-message.md), однако отличие заключается в том, что оно действует только для текущего соединения, то есть можно установить обратный вызов onMessage для конкретного соединения.

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// При событии подключения клиента
$worker->onConnect = function(TcpConnection $connection)
{
    // Установка обратного вызова onMessage для соединения
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('receive success');
    };
};
// Запуск воркера
Worker::runAll();
```

Вышеприведенный код эквивалентен следующему

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Прямая установка обратного вызова onMessage для всех соединений
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// Запуск воркера
Worker::runAll();
```
