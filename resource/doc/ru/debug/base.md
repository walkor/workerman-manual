# Основная отладка

У WorkerMan есть два режима работы: отладочный режим и режим демона.

Запустите `php start.php start` для входа в отладочный режим, где вывод функций `echo, var_dump, var_export` будет отображаться в терминале. Обратите внимание, что при запуске WorkerMan с помощью `php start.php start` все процессы завершатся при закрытии терминала.

Если вы запустите `php start.php start -d`, то это будет режим демона, который используется для реального онлайн-режима и не зависит от закрытия терминала.

Если вы хотите видеть вывод функций, таких как `echo, var_dump, var_export` при запуске в режиме демона, вы можете установить свойство Worker::$stdoutFile, как показано ниже:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Вывод на экран записывается в указанный файл Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

Таким образом, все выводы функций `echo, var_dump, var_export` будут записываться в файл, указанный в `Worker::$stdoutFile`. Обратите внимание, что указанный путь `Worker::$stdoutFile` должен иметь права на запись.
