# globalEvent

## Описание:
```php
static Event Worker::$globalEvent
```

Этот атрибут является статическим глобальным, представляет собой глобальный экземпляр eventloop, в который можно зарегистрировать события чтения или записи файлового дескриптора, а также события сигналов.

## Пример

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'Pid is ' . posix_getpid() . "\n";
    // Когда процесс получает сигнал SIGALRM, вывести некоторую информацию
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Get signal SIGALRM\n";
    });
};
// Запуск worker'а
Worker::runAll();
```

## Тест
После запуска Workerman будет выведен идентификатор текущего процесса (число). Запустите следующую команду в командной строке:
``` 
kill -SIGALRM идентификатор_процесса
```
На сервере будет выведено:
```
Get signal SIGALRM
```
