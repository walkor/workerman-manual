# reloadable
## Описание:
```php
bool Worker::$reloadable
```

При выполнении команды `php start.php reload` будет отправлен сигнал перезагрузки (SIGUSR1) всем дочерним процессам.

После получения сигнала перезагрузки дочерний процесс автоматически завершится, после чего основной процесс автоматически запустит новый процесс, обычно используется для обновления бизнес-логики.

Когда свойство $reloadable процесса равно false, после получения сигнала перезагрузки будет только вызвано [onWorkerReload](on-worker-reload.md), но текущий процесс не будет перезапущен.

Например, процесс gateway в модели Gateway/Worker отвечает за поддержание соединения с клиентом, а процесс worker отвечает за обработку запросов.
Установив свойство reloadable процесса gateway в false, можно обновить бизнес-логику без разрыва соединения с клиентом во время перезагрузки.

## Пример

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Установка возможности перезапуска для данного экземпляра
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Запуск рабочего процесса...\n";
};
// Запуск worker
Worker::runAll();
```
