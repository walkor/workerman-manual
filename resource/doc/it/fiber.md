# Coroutine Fiber
A partire da workerman 5.0.0, è supportato il [Coroutine Fiber](https://www.php.net/manual/zh/language.fibers.php).

> **Nota**
> La funzionalità Fiber richiede PHP >= 8.1 e l'installazione di `composer require revolt/event-loop ^1.0.0`.

### Introduzione

Fiber è una coroutine (fibra) integrata in PHP, in grado di interrompere il codice PHP e poi riprenderne l'esecuzione quando necessario. Il suo maggior vantaggio è consentire agli sviluppatori di scrivere codice asincrono non bloccante in modo sincrono, migliorando notevolmente la manutenibilità del codice.

### Esempio
Di seguito viene mostrato un esempio per confrontare la differenza tra la programmazione asincrona con callback e le coroutine.
Supponiamo di dover chiamare un'interfaccia HTTP e poi ritardare la risposta di un secondo. Ecco come sarebbero i due approcci con callback asincrono e con coroutine.

**Utilizzo del callback asincrono**
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
    // Chiamata all'interfaccia HTTP
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Invia dopo un secondo di ritardo
        Timer::add(1, function() use ($connection, $response) {
            // Invia i dati al browser
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Utilizzo della coroutine**
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
    // Chiamata all'interfaccia HTTP
    $response = $http->get('http://example.com/');
    // Ritardo di 1 secondo
    Timer::sleep(1);
    // Invia i dati
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Nota**
> Il codice sopra richiede l'installazione di `composer require workerman/http-client ^2.0.0`.

Entrambi questi approcci eseguono operazioni asincrone non bloccanti e hanno un'efficienza operativa elevata, ma l'utilizzo delle coroutine è più leggibile e più facile da mantenere rispetto ai callback asincroni.

### Considerazioni sulle Fiber
* Le Fiber coroutine non supportano la creazione di coroutine per Pdo, Redis e le funzioni di blocco interne di PHP, il che significa che l'utilizzo di queste estensioni e funzioni rimane comunque bloccante.
* Attualmente i client Fiber coroutine disponibili sono [workerman/http-client](../components/workerman-http-client.md) e [workerman/redis](../components/workerman-redis.md).

# Coroutine Swoole
A partire da workerman v5, è supportata l'integrazione di Swoole come motore di eventi sottostante.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// È necessario impostare manualmente Swoole come motore di eventi sottostante
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
**Suggerimenti**
* Si consiglia l'uso di swoole5.0 o versioni successive.
* L'utilizzo di Swoole come motore di eventi sottostante consente a workerman di supportare le coroutine di Swoole.
* Quando si utilizza Swoole come motore di eventi sottostante, non è necessario installare l'estensione event.
* Swoole di default non ha abilitato le coroutine one-click, il che significa che le chiamate bloccanti basate su Pdo, Redis, lettura/scrittura di file incorporati in PHP rimangono bloccanti.
* Per attivare le coroutine one-click, è necessario chiamare manualmente `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL)`.

Per ulteriori informazioni, consulta il [manuale di Swoole](https://wiki.swoole.com/).

Per ulteriori informazioni, consulta [Event Driven](appendices/event.md).

# Riguardo alle coroutine
Innanzitutto, non bisogna essere eccessivamente devoti alle coroutine. Quando i database, Redis e altri archivi sono all'interno della rete, spesso le chiamate bloccanti multi-processo sono più veloci delle coroutine. Secondo i [dati di benchmark degli ultimi 3 anni su techempower.com](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db), le prestazioni delle chiamate bloccanti al database con workerman sono superiori alle connessioni del pool del database di swoole con coroutine, addirittura quasi il doppio rispetto ai framework adottanti coroutine come go language gin, echo, etc.

workerman ha già aumentato le prestazioni delle applicazioni PHP di diverse volte e persino di decine di volte. Per la stragrande maggioranza dei progetti workerman, l'aggiunta di coroutine potrebbe non portare a un miglioramento significativo delle prestazioni.
Se ci sono chiamate lente all'interno del sistema, come ad esempio chiamate HTTP esterne, è possibile considerare l'utilizzo delle coroutine per migliorare le prestazioni.
