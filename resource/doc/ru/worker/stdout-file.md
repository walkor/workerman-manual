# stdoutFile
## Описание:
```php
static string Worker::$stdoutFile
```

Это статическое свойство доступно глобально. Если процесс запущен в режиме демона (с флагом ```-d```), то все выводы в терминал (такие как echo или var_dump) будут перенаправлены в файл, указанный в stdoutFile.

Если не указано, и процесс запущен в режиме демона, то весь терминальный вывод будет перенаправлен в `/dev/null` (то есть по умолчанию все выводы будут отбрасываться).

> Примечание: `/dev/null` - это специальный файл в Linux, который фактически является черной дырой, все данные, записанные в этот файл, будут отброшены. Если вы не хотите потерять вывод, вы можете установить `Worker::$stdoutFile` в путь к обычному файлу.

> Примечание: Это свойство должно быть установлено до запуска ```Worker::runAll();```. Эта функция не поддерживается в системах Windows.

## Пример

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Весь вывод будет сохранен в файле /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Запуск worker
Worker::runAll();
```
