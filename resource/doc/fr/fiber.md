# Les coroutines Fiber
Depuis la version 5.0.0, Workerman prend en charge les [coroutines Fiber](https://www.php.net/manual/zh/language.fibers.php).

> **Remarque**
> Les fonctionnalités Fiber nécessitent PHP>=8.1 et l'installation de `composer require revolt/event-loop ^1.0.0`.

### Introduction

Fiber est une coroutine (ou fibre) intégrée à PHP, qui permet d'interrompre le code PHP puis de le reprendre au besoin. Son principal avantage est de permettre aux développeurs d'écrire du code asynchrone non bloquant de manière synchrone, ce qui améliore considérablement la maintenabilité du code.

### Exemple
Voici un exemple comparant l'utilisation des coroutines avec la programmation asynchrone basée sur des rappels. Supposons que nous ayons besoin d'appeler une API HTTP et de différer la réponse d'une seconde. Voici les exemples de code pour l'approche asynchrone avec rappels et l'approche coroutine.

**Approche asynchrone avec rappels**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Appel de l'API HTTP
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Différer l'envoi d'une seconde
        Timer::add(1, function() use ($connection, $response) {
            // Envoyer les données au navigateur
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Approche coroutine**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Appeler l'API HTTP
    $response = $http->get('http://example.com/');
    // Différer d'1 seconde
    Timer::sleep(1);
    // Envoyer les données
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Remarque**
> Ces deux méthodes sont des exécutions asynchrones non bloquantes, et sont toutes deux très efficaces en termes d'exécution. Cependant, l'approche coroutine est plus facile à lire et davantage propice à la maintenance que l'approche asynchrone avec rappels.

### Remarques sur les coroutines Fiber
* Les coroutines Fiber ne prennent pas en charge la coroutinisation des fonctions de blocage des extensions Pdo, Redis et des fonctions de blocage internes de PHP. Cela signifie que l'utilisation de ces extensions et fonctions reste un appel bloquant.
* Les clients de coroutines Fiber actuellement disponibles incluent [workerman/http-client](../components/workerman-http-client.md) et [workerman/redis](../components/workerman-redis.md).

# Coroutines avec Swoole
Workerman v5 prend en charge l'utilisation de Swoole comme pilote d'événements sous-jacent.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Vous devez définir manuellement Swoole comme pilote d'événements sous-jacent
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
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
**Remarque**
* Il est recommandé d'utiliser Swoole 5.0 ou une version ultérieure.
* Utiliser Swoole comme pilote d'événements sous-jacent permet à Workerman de supporter les coroutines de Swoole.
* Lorsque Swoole est utilisé comme pilote d'événements sous-jacent, il n'est pas nécessaire d'installer l'extension event.
* Par défaut, Swoole n'active pas les coroutines en un clic, ce qui signifie que les appels bloquants basés sur Pdo, Redis, la lecture/écriture de fichiers internes à PHP sont utilisés de manière bloquante.
* Pour activer les coroutines en un clic, vous devez appeler manuellement `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL)`.

Pour plus d'informations, veuillez consulter le [manuel de Swoole](https://wiki.swoole.com/). 

Pour en savoir plus, veuillez consulter [l'événement piloté](appendices/event.md).

# À propos des coroutines
Il n'est pas nécessaire de surestimer les coroutines. Lorsque les bases de données, Redis et autres systèmes de stockage sont utilisés en interne, les appels bloquants dans des processus multiples sont souvent plus rapides que les coroutines. Selon les [données de test TechEmpower des trois dernières années](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db), les performances des appels bloquants à la base de données avec Workerman sont meilleures que les connexions de pool avec coroutines de Swoole, et même environ 1 fois supérieures à celles des frameworks à base de coroutines tels que gin et echo en Go.

Workerman a déjà augmenté les performances des applications PHP de plusieurs fois, voire plusieurs dizaines de fois. Pour la plupart des projets Workerman, l'ajout de coroutines n'apportera pas une amélioration significative des performances.
Si vous avez des appels lents dans votre système, tels que des appels HTTP externes, vous pouvez envisager d'utiliser des coroutines pour augmenter les performances.
