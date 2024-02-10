# имя

## Описание:
```php
string Worker::$name
```

Устанавливает имя текущего экземпляра Worker для удобства идентификации процесса при запуске команды status. По умолчанию не установлено и равно "none".

## Пример

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Установка имени экземпляра
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Запуск рабочего...\n";
};
// Запуск рабочего
Worker::runAll();
```
