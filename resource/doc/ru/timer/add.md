```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Регулярно выполнять определенную функцию или метод класса.

Примечание: Таймер выполняется в текущем процессе, в workerman не создаются новые процессы или потоки для выполнения таймера.

### Параметры
 ``` time_interval ```
 
Как часто выполнять, в секундах, допускаются десятичные значения, можно указывать точность до 0.001, то есть до миллисекунд.


 ``` callback ```
 
Функция обратного вызова. Обратите внимание: Если функция обратного вызова является методом класса, то этот метод должен быть публичным.


 ``` args ```
 
Параметры функции обратного вызова, должны быть массивом, элементы массива - это значения параметров.


 ``` persistent ```
 
Является ли таймер постоянным. Если нужно выполнить таймер только один раз, то передайте false (задача, которая должна выполниться только один раз, автоматически уничтожается после выполнения, не требуется вызывать `Timer::del()`). По умолчанию true, таймер будет регулярно выполняться.

### Возвращаемое значение
Возвращает целое число, представляющее идентификатор таймера, который можно уничтожить, вызвав `Timer::del($timerid)`.

### Примеры

#### 1. Регулярно выполнять анонимную функцию (замыкание)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Запускать задачу в нескольких процессах, обратите внимание на возможные проблемы с конкурентностью работы приложения в нескольких процессах
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Выполнять каждые 2.5 секунды
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "task run\n";
    });
};

// Запуск worker'а
Worker::runAll();
```

### 2. Установить таймер только в определенном процессе

У экземпляра worker'а есть 4 процесса, устанавливаем таймер только в процессе с идентификатором 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    // Устанавливаем таймер только в процессе с идентификатором 0, остальные процессы (1, 2, 3) таймер не имеют
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 worker процесса, таймер установлен только в 0 процессе\n";
        });
    }
};
// Запуск worker'а
Worker::runAll();
```

### 3. Регулярно выполнять анонимную функцию, использование замыкания для передачи параметров
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// Установить таймер для соответствующего соединения при установлении соединения
$ws_worker->onConnect = function(TcpConnection $connection)
{
    // Выполнять каждые 10 секунд
    $time_interval = 10;
    $connect_time = time();
    // Временно добавить свойство timer_id объекту соединения для сохранения идентификатора таймера
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};
// При закрытии соединения удаляем соответствующий таймер
$ws_worker->onClose = function(TcpConnection $connection)
{
    // Удалить таймер
    Timer::del($connection->timer_id);
};

// Запуск worker'а
Worker::runAll();
```
