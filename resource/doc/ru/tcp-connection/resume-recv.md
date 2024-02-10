# resumeRecv
## Описание:
```php
void Connection::resumeRecv(void)
```

Продолжает прием данных в текущем соединении. Этот метод полезен для управления потоком данных при передаче данных.

## Параметры

Без параметров

## Пример

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // Добавляем динамическое свойство объекту connection для сохранения количества запросов в текущем соединении
    $connection->messageCount = 0;
};
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // После приема 100 запросов в соединении, данные больше не принимаются
    $limit = 100;
    if(++$connection->messageCount > $limit)
    {
        $connection->pauseRecv();
        // Восстановление приема данных через 30 секунд
        Timer::add(30, function($connection){
            $connection->resumeRecv();
        }, array($connection), false);
    }
};
// Запуск worker
Worker::runAll();
```

## См. также
void Connection::pauseRecv(void) - Останавливает прием данных для соответствующего объекта соединения
