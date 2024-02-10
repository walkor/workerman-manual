# stopAll
```php
void Worker::stopAll(void)
```

Останавливает текущий процесс и выходит из него.

> **Примечание**
> `Worker::stopAll()` используется для остановки текущего процесса. После выхода из текущего процесса главный процесс сразу же запустит новый процесс. Если вы хотите остановить всю службу workerman, вызовите `posix_kill(posix_getppid(), SIGINT)`

### Параметры
Без параметров

### Возвращаемое значение
Без возврата

## Пример max_request

В следующем примере дочерний процесс выполняет stopAll и выходит после обработки 1000 запросов для перезапуска нового процесса. Это аналогично свойству max_request в php-fpm и используется в основном для решения проблем утечки памяти, вызванных ошибками в бизнес-коде php.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Каждый процесс обрабатывает не более 1000 запросов
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Количество обработанных запросов
    static $request_count = 0;

    $connection->send('hello http');
    // Если количество запросов достигло 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Выход из текущего процесса, главный процесс сразу же запустит новый процесс для завершения перезапуска процесса
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
