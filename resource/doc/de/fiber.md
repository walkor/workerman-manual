# Fiber
Ab Version 5.0.0 unterstützt Workerman [Fiber-Co-Routinen (Fasern)](https://www.php.net/manual/zh/language.fibers.php).

> **Hinweis**
> Die Fiber-Funktion erfordert PHP >= 8.1 und die Installation von `composer require revolt/event-loop ^1.0.0`.

### Einführung
Fiber ist eine eingebaute Co-Routine (Faser) in PHP, die den PHP-Code unterbrechen und bei Bedarf wieder aufnehmen kann. Seine größte Wirkung besteht darin, dass Entwickler asynchronen, nicht blockierenden Code synchron schreiben können, was die Wartbarkeit des Codes erheblich verbessert.

### Beispiel
Im Folgenden werden die Unterschiede zwischen Co-Routinen und asynchronem Rückrufcode anhand eines Beispiels erläutert. Angenommen, wir müssen eine Anforderung erfüllen, die es erfordert, eine HTTP-Schnittstelle aufzurufen und dann eine Verzögerung von einer Sekunde zu erzeugen. Die Implementierungen für asynchrone Rückrufe und Co-Routinen lauten wie folgt:

**Implementierung mit asynchronem Rückrufcode**
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
    // Anfrage an die HTTP-Schnittstelle
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Verzögerung von einer Sekunde hinzufügen
        Timer::add(1, function() use ($connection, $response) {
            // Daten an den Browser senden
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Implementierung mit Co-Routinen**
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
    // Aufruf der HTTP-Schnittstelle
    $response = $http->get('http://example.com/');
    // Verzögerung von 1 Sekunde
    Timer::sleep(1);
    // Daten senden
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Hinweis**
> Der obige Code erfordert die Installation von `composer require workerman/http-client ^2.0.0`.

Beide Implementierungen sind asynchron und blockieren nicht, wobei die Effizienz des Betriebs sehr hoch ist. Allerdings ist die Implementierung mit Co-Routinen lesbarer und wartungsfreundlicher als asynchrone Rückrufe.

### Fiber Hinweise
* Fiber-Co-Routinen unterstützen nicht die Ko-Routinisierung von Pdo, Redis und internen blockierenden Funktionen von PHP. Das bedeutet, dass die Verwendung dieser Erweiterungen und Funktionen nach wie vor blockierende Aufrufe erfordert.
* Derzeit verfügbare Fiber-Co-Routine-Client umfassen [workerman/http-client](../components/workerman-http-client.md) und [workerman/redis](../components/workerman-redis.md).

# Swoole Co-Routinen
Workerman v5 unterstützt auch die Verwendung von Swoole als unterliegendes Ereignis-Backend.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Hier muss Swoole manuell als unterliegendes Ereignis-Backend festgelegt werden
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

**Hinweis**
* Es wird empfohlen, Swoole 5.0 oder eine höhere Version zu verwenden.
* Die Verwendung von Swoole als unterliegendes Ereignis-Backend ermöglicht es Workerman, Swooles Co-Routinen zu unterstützen.
* Bei Verwendung von Swoole als unterliegendes Ereignis-Backend ist keine Installation der Event-Erweiterung erforderlich.
* Swoole hat standardmäßig keine Ein-Klick-Co-Routinen aktiviert, was bedeutet, dass die Verwendung von Pdo, Redis und den internen Datei-Inputs/Ausgaben von PHP blockierende Aufrufe erfordert.
* Um Ein-Klick-Co-Routinen zu aktivieren, muss `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);` manuell aufgerufen werden.

Weitere Informationen finden Sie im [Swoole-Handbuch](https://wiki.swoole.com/).

Weitere Informationen finden Sie unter [Ereignis-Backend](appendices/event.md).

# Über Co-Routinen
Zunächst einmal sollte man Co-Routinen nicht übermäßig verehren. Wenn Datenbanken, Redis und andere Speicher in einem lokalen Netzwerk sind, ist in vielen Fällen die blockierende Ausführung mit mehreren Prozessen schneller als Co-Routinen. Anhand der Testdaten von [techempower.com der letzten 3 Jahre](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db) geht hervor, dass die Leistungsfähigkeit der blockierenden Datenbankaufrufe in Workerman höher ist als die von Swoole-Datenbankverbindungen mit Pool und Co-Routinen und sogar fast doppelt so hoch wie die von Co-Routinen-Frameworks wie Gin und Echo in der Go-Sprache.

Workerman hat die Leistungsfähigkeit von PHP-Anwendungen um das Vielfache erhöht. In den meisten Workerman-Projekten wird die Leistung durch die Hinzufügung von Co-Routinen wahrscheinlich nicht wesentlich verbessert. Wenn Ihr System langsame Aufrufe enthält, wie externe HTTP-Aufrufe, kann die Verwendung von Co-Routinen zur Leistungssteigerung in Betracht gezogen werden.
