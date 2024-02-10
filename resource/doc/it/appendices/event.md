# Eventi supportati attualmente da Workerman

| Nome | Dipendenze esterne | Supporto per coroutine | Priorità | Versione di Workerman |
| ----- | ------ | -- | ----- |
| Workerman\Events\Select | Nessuna | Non supportato | Supportato di default nel kernel | >=3.0 |
| Workerman\Events\Revolt | event (opzionale) | Supportato | Richiede l'installazione di [revolt/event-loop](https://github.com/revoltphp/event-loop) | >=5.0 |
| Workerman\Events\Event | event | Non supportato | Supportato di default nel kernel | >=3.0 |
| Workerman\Events\Swoole | [swoole](https://github.com/swoole/swoole-src) | Supportato | Richiede configurazione manuale | >=4.0 |
| Workerman\Events\Swow | [swow](https://github.com/swow/swow) | Supportato | Richiede configurazione manuale | >=5.0 |

* Ogni driver del kernel fornisce funzionalità specifiche. Ad esempio, l'uso di `Revolt` consente a Workerman di supportare le [Fiber coroutine](https://www.php.net/manual/zh/language.fibers.php) integrate in PHP, mentre l'uso di `Swoole` consente a Workerman di supportare le coroutine di Swoole.
* I diversi driver degli eventi sono mutualmente esclusivi. Ad esempio, non è possibile utilizzare le coroutine di Swoole o Swow quando si utilizza `Revolt` e le sue Fiber coroutine.
* Per utilizzare `Revolt`, è necessario installare `composer require revolt/event-loop ^1.0.0`. Dopo l'installazione, il kernel di Workerman imposterà automaticamente `Revolt` come driver degli eventi preferito.
* `Swoole` e `Swow` richiedono di impostare manualmente `Worker::$eventLoopClass` per essere attivati (vedi prossimo paragrafo).
* Di default, swoole non abilita il [Runtime con coroutine](https://wiki.swoole.com/#/runtime?id=runtime), il che significa che le chiamate basate su Pdo, Redis e sulle operazioni di lettura/scrittura dei file integrati in PHP sono ancora bloccanti.
* Per abilitare il runtime con coroutine in swoole, è necessario chiamare manualmente `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`.

> **Nota**
> L'estensione swow cambierà automaticamente il comportamento di alcune funzioni integrate in PHP, il che può impedire a Workerman di gestire le richieste e i segnali se swow è abilitato ma non utilizzato come driver degli eventi. Quindi, se non si utilizza swow come driver di base, è necessario commentare swow da php.ini.

Per ulteriori informazioni, consultare [Workerman coroutine](../fiber.md).

# Impostare manualmente il driver degli eventi per Workerman

Ecco come impostare manualmente il driver degli eventi per Workerman:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Impostare manualmente il driver degli eventi di base
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
