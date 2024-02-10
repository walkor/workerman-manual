# pidFile
## Описание:
```php
static string Worker::$pidFile
```

По возможности рекомендуется не устанавливать это свойство.

Это глобальное статическое свойство используется для установки пути файла pid процесса WorkerMan.

Этот параметр полезен для мониторинга, например, помещая файл pid WorkerMan в фиксированный каталог, можно удобно читать файл pid некоторым программным обеспечением для мониторинга состояния процессов WorkerMan.

По умолчанию WorkerMan будет автоматически создавать файл pid в каталоге, параллельном каталогу Workerman (обратите внимание, что до версии workerman 3.2.3 по умолчанию файл pid создавался в ```sys_get_temp_dir()```), чтобы избежать конфликта pid при запуске нескольких экземпляров WorkerMan, файл pid WorkerMan содержит путь к текущему экземпляру WorkerMan.

Примечание: это свойство должно быть установлено до запуска ```Worker::runAll();``` для того, чтобы оно было действительным. Эта функция не поддерживается в системе Windows.


## Пример

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Запуск worker
Worker::runAll();
```
