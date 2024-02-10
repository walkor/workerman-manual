# Unterstützte ereignisgesteuerte Treiber in Workerman

| Name  | Abhängige Erweiterung | Unterstützung von Koroutinen |  Priorität  |  Workerman-Version  |
|-----|------|--|-----|
|  Workerman\Events\Select   |   Keine   | Nicht unterstützt  |  Standardmäßig vom Kernel unterstützt   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event (optional)   | Unterstützt |  Erfordert die Installation von [revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | Nicht unterstützt |  Standardmäßig vom Kernel unterstützt   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | Unterstützt |  Erfordert manuelle Konfiguration   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | Unterstützt |  Erfordert manuelle Konfiguration   |  >=5.0  |

* Jeder Kernel-Treiber bietet individuelle Funktionen, z.B. die Verwendung von `Revolt` ermöglicht die Unterstützung von PHP-internen [Fiber-Koroutinen](https://www.php.net/manual/zh/language.fibers.php) in Workerman, während die Verwendung von `Swoole` die Unterstützung von Swoole-Koroutinen ermöglicht.
* Die verschiedenen Ereignistreiber sind sich gegenseitig ausschließend. Zum Beispiel kann bei Verwendung von `Revolt` mit Fiber-Koroutinen nicht auf die Koroutinen von Swoole oder Swow zugegriffen werden.
* Zum Installieren von `Revolt` ist `composer require revolt/event-loop ^1.0.0` erforderlich. Nach der Installation wird der Workerman-Kernel automatisch als bevorzugter Ereignistreiber eingestellt.
* `Swoole` und `Swow` müssen manuell mit `Worker::$eventLoopClass` konfiguriert werden, um wirksam zu werden (siehe folgender Absatz).
* Swoole aktiviert standardmäßig kein [Ein-Klick-Koroutine-Runtime](https://wiki.swoole.com/#/runtime?id=runtime), was bedeutet, dass PDO-, Redis- und PHP-interner Datei-Lese-/Schreibvorgänge immer noch blockierend sind.
* Um Swoole für eine Ein-Klick-Koroutine zu aktivieren, muss manuell `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` aufgerufen werden.

> **Hinweis**
> Die Swow-Erweiterung ändert automatisch das Verhalten einiger PHP-interner Funktionen. Dadurch kann es vorkommen, dass Workerman bei aktivierter Swow-Erweiterung, aber ohne Verwendung von Swow als Ereignistreiber, keine Anfragen oder Signale beantworten kann. Wenn Sie also Swow nicht als den zugrunde liegenden Treiber verwenden, müssen Sie Swow aus der php.ini-Konfigurationsdatei auskommentieren.

Weitere Informationen finden Sie unter [Workerman-Koroutinen](../fiber.md).

# Einrichten von Ereignistreibern für Workerman

Nachstehend finden Sie die manuelle Konfiguration der Ereignistreiber für Workerman:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Manuelle Festlegung des zugrunde liegenden Ereignistreibers
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
