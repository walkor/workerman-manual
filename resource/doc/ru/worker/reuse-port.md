# reusePort
> **Примечание**
> Требуется workerman>= 3.2.1 PHP>=7.0, Операционные системы Windows и Mac OS не поддерживают эту функцию.

## Описание:

```php
bool Worker::$reusePort
```

Устанавливает, открыт ли текущий рабочий процесс для повторного использования порта прослушивания (опция SO_REUSEPORT сокета).

После включения повторного использования порта прослушивания разрешается нескольким независимым процессам прослушивать один и тот же порт, и ядро системы балансирует нагрузку, решая, какой процесс будет обрабатывать соединение с сокетом, избегая эффекта стада, что может улучшить производительность многопроцессовых приложений с короткими соединениями.

**Примечание:** Для этой функции требуется версия PHP >= 7.0

## Пример 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Запуск рабочего процесса
Worker::runAll();
```

## Пример 2: прослушивание нескольких портов (различные протоколы) в workerman
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// После запуска каждого процесса добавляем прослушивание в текущем процессе
$worker->onWorkerStart = function ($worker) {
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Несколько процессов прослушивают один и тот же порт (слушающий сокет не наследуется от родительского процесса)
     * Необходимо включить повторное использование порта, иначе будет ошибка "Address already in use"
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Выполнение прослушивания
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Запуск рабочего процесса
Worker::runAll();
```
