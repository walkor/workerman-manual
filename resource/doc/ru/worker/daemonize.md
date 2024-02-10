# Демонизация
## Описание:
```php
static bool Worker::$daemonize
```

Это глобальный статический атрибут, который указывает, запущен ли процесс в режиме демона. Если запуск производится с помощью параметра ```-d```, то этот атрибут автоматически устанавливается в значение true. Его также можно установить вручную в коде.

Примечание: этот атрибут должен быть установлен до запуска ```Worker::runAll();``` для того, чтобы он действовал. Эта функция не поддерживается в операционных системах Windows.

## Пример:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Запуск воркера\n";
};
// Запуск воркера
Worker::runAll();
```
