# pauseRecv
## Описание:
```php
void Connection::pauseRecv(void)
```

Останавливает прием данных в текущем соединении. Обратный вызов onMessage для этого соединения не будет вызван. Этот метод очень полезен для управления потоком передачи данных.

## Параметры

Без параметров

## Пример

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    // Динамически добавляем свойство объекту connection для сохранения количества запросов, поступивших по текущему соединению
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // После получения 100 запросов для каждого соединения остановить прием данных
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
    }
};
// Запуск worker
Worker::runAll();
```

## Смотрите также
void Connection::resumeRecv(void) - возобновляет прием данных для соответствующего объекта соединения
