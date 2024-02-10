```php
boolean \Workerman\Timer::del(int $timer_id)
```
Удаление определенного таймера

### Параметры
``` timer_id ```

ID таймера, возвращенный функцией add, в виде целого числа

### Возвращаемое значение
boolean

### Пример
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Запуск задачи в нескольких процессах. Обратите внимание на проблемы совместного использования ресурсов при многопоточной работе
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Запуск каждые 2 секунды
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // Запуск одноразовой задачи через 20 секунд и удаление периодической задачи с интервалом в 2 секунды
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Запуск воркера
Worker::runAll();
```

### Пример (удаление текущего таймера в обратном вызове таймера)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Обратите внимание, что для использования ID текущего таймера в обратном вызове нужно использовать ссылку (&)
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // Удаление таймера после 10 выполнений
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Запуск воркера
Worker::runAll();
```
