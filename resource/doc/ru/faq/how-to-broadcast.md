# Как отправить широковещательные (групповые) данные

## Пример (регулярная рассылка)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// В этом примере количество процессов должно быть равно 1
$worker->count = 1;
// При запуске процесса устанавливаем таймер для регулярной отправки данных всем клиентам
$worker->onWorkerStart = function($worker)
{
    // Регулярная отправка данных каждые 10 секунд
    Timer::add(10, function()use($worker)
    {
        // Перебираем все соединения текущего процесса и отправляем текущее время сервера
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Запуск worker
Worker::runAll();
```

## Пример (групповой чат)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// В этом примере количество процессов должно быть равно 1
$worker->count = 1;
// При получении сообщения от клиента, отправляем его всем остальным пользователям
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Запуск worker
Worker::runAll();
```

## Пояснение:
**Одиночный процесс:**
Приведенные выше примеры могут работать только **с одним процессом** (```$worker->count=1```), потому что при многопроцессорном запуске различные клиенты могут подключаться к разным процессам, и клиенты из разных процессов будут изолированы друг от друга и не смогут напрямую общаться. Например, процесс A не сможет **напрямую** управлять объектами connection клиентов процесса B (для этого требуется межпроцессорное взаимодействие, такое как использование компонента Channel, например [Пример-кластерная отправка](../components/channel-examples.md), [Пример-групповая отправка](../components/channel-examples2.md)).

**Рекомендуется использовать GatewayWorker**
GatewayWoker, разработанный на основе Workerman, предоставляет более удобный механизм рассылки, включая групповые сообщения и широковещательные рассылки, а также позволяет настраивать многопроцессорное и даже многосерверное развертывание. Если требуется отправлять данные клиентам, рекомендуется использовать фреймворк GatewayWorker.

Ссылка на руководство по GatewayWorker: https://doc2.workerman.net
