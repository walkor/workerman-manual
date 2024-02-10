# Поддерживаемые события для workerman

| Название | Зависимость | Поддержка корутин | Приоритет | Версия workerman |
|-----|------|--|-----|
| Workerman\Events\Select | Нет | Не поддерживается | Поддерживается по умолчанию | >=3.0 |
| Workerman\Events\Revolt | event (необязательно) | Поддерживается | Требуется установка [revolt/event-loop](https://github.com/revoltphp/event-loop) | >=5.0 |
| Workerman\Events\Event | event | Не поддерживается | Поддерживается по умолчанию | >=3.0 |
| Workerman\Events\Swoole | [swoole](https://github.com/swoole/swoole-src) | Поддерживается | Требуется ручная настройка | >=4.0 |
| Workerman\Events\Swow | [swow](https://github.com/swow/swow) | Поддерживается | Требуется ручная настройка | >=5.0 |

* Каждый тип ядра будет предоставлять отдельные функции, например, использование `Revolt` позволит workerman поддерживать встроенные в PHP [Fiber корутины](https://www.php.net/manual/zh/language.fibers.php), а использование `Swoole` позволит workerman поддерживать корутины Swoole
* Между различными событиями существует взаимоисключающее отношение, например, при использовании `Revolt` для корутин Fiber нельзя использовать корутины Swoole или Swow
* Для использования `Revolt` необходимо установить `composer require revolt/event-loop ^1.0.0`, после установки этого workerman автоматически установит его в качестве предпочтительного событийного драйвера
* Для использования `Swoole` и `Swow` необходимо ручно установить `Worker::$eventLoopClass` (см. следующий абзац)
* По умолчанию в swoole не включается [однопоточный режим Runtime](https://wiki.swoole.com/#/runtime?id=runtime), что означает, что вызовы на основе Pdo, Redis, встроенной в PHP файловой системы всё ещё являются блокирующими
* Для включения однопоточного режима в swoole необходимо вручную вызвать `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

> **Заметка**
> Расширение swow автоматически изменяет поведение некоторых встроенных функций PHP, что может привести к тому, что при использовании swow в качестве драйвера событий workerman не сможет реагировать на запросы и сигналы, поэтому, если вы не используете swow в качестве основной драйвера, вам следует закомментировать swow в php.ini

Больше информации см. [workerman корутины](../fiber.md)

# Настройка драйвера событий для workerman

Ниже приведен пример ручной настройки драйвера событий для workerman

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Установка драйвера событий вручную
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```
