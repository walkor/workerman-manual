# logFile
## Описание:
```php
static string Worker::$logFile
```

Используется для указания местоположения файла журнала Workerman.

В этом файле записываются связанные с Workerman события, такие как запуск, остановка и т.д.

Если не установлен явно, имя файла по умолчанию - workerman.log, а расположение файла находится в родительском каталоге Workerman.

**Примечание:**

В этом журнальном файле записываются только события, связанные с запуском или остановкой Workerman, но не содержится никаких бизнес-журналов.

Для бизнес-журналов можно использовать функции [file_put_contents](https://php.net/manual/zh/function.file-put-contents.php) или [error_log](https://php.net/manual/zh/function.error-log.php) для собственной реализации.

## Пример

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Запуск воркера";
};
// Запуск воркера
Worker::runAll();
```
