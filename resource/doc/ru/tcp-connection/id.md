# id

## Описание:
```php
int Connection::$id
```

Идентификатор соединения. Это увеличивающееся целое число.

Примечание: workerman является многопроцессовым, внутри каждого процесса поддерживается увеличивающийся идентификатор соединения, поэтому идентификаторы соединений могут повторяться между несколькими процессами.
Если требуется уникальный идентификатор соединения, можно присвоить новое значение свойству connection->id по необходимости, например, добавив префикс worker->id.

## См. также
[Свойство connections объекта Worker](../worker/connections.md)

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo $connection->id;
};
// Запуск worker
Worker::runAll();
```
