# Les événements pris en charge par workerman actuellement

| Nom | Extension requise | Support de coroutine | Priorité | Version de workerman |
|-----|------|--|-----|
|  Workerman\Events\Select   |   Aucune   | Non pris en charge  |  Prise en charge par défaut du noyau   |  >=3.0  ｜
|  Workerman\Events\Revolt   |   event (optionnel)   | Pris en charge |  Nécessite l'installation de [revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | Non pris en charge |  Prise en charge par défaut du noyau   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | Pris en charge |  Nécessite une configuration manuelle   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | Pris en charge |  Nécessite une configuration manuelle   |  >=5.0  |

* Chaque pilote de noyau offre des fonctionnalités spécifiques, par exemple, l'utilisation de `Revolt` permet à workerman de prendre en charge les [coroutines (fibres)](https://www.php.net/manual/zh/language.fibers.php) intégrées à PHP, tandis que l'utilisation de `Swoole` permet à workerman de prendre en charge les coroutines de Swoole.
* Les différents pilotes d'événements sont mutuellement exclusifs. Par exemple, l'utilisation de la coroutines `Revolt` de Fiber empêche l'utilisation des coroutines de Swoole ou Swow.
* `Revolt` nécessite l'installation de `composer require revolt/event-loop ^1.0.0`. Une fois installé, le noyau de workerman le configure automatiquement comme pilote d'événements préféré.
* `Swoole` et `Swow` doivent être configurés manuellement avec `Worker::$eventLoopClass` pour être pris en compte (voir le paragraphe suivant).
* Par défaut, Swoole n'active pas [Runtime coroutine en un clic](https://wiki.swoole.com/#/runtime?id=runtime), ce qui signifie que les appels basés sur Pdo, Redis, et les opérations de lecture/écriture de fichiers intégrés à PHP restent bloquants.
* Pour activer le Runtime Coroutine en un clic de Swoole, il est nécessaire d'appeler manuellement `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`.

> **Remarque**
> L'extension swow modifie automatiquement le comportement de certaines fonctions intégrées à PHP, ce qui empêche workerman de répondre aux demandes et signaux lorsqu'elle est activée sans être utilisée comme pilote d'événements. Par conséquent, si vous n'utilisez pas swow comme pilote de bas niveau, vous devez commenter swow dans php.ini.

Voir aussi [Coroutines de workerman](../fiber.md) pour plus de détails.

# Configuration manuelle du pilote d'événements pour workerman

Voici comment configurer manuellement le pilote d'événements pour workerman :

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Configuration manuelle du pilote d'événements pour workerman
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
