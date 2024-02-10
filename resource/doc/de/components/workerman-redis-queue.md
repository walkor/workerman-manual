# workerman/redis-queue

Eine auf Redis basierende Nachrichtenwarteschlange mit Unterstützung für die Verarbeitung verzögerter Nachrichten.

## Projektadresse:
https://github.com/walkor/redis-queue

## Installation:
```
composer require workerman/redis-queue
```

## Beispiel
```
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\RedisQueue\Client;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $client = new Client('redis://127.0.0.1:6379');
   // Abonnement
    $client->subscribe('user-1', function($data){
        echo "user-1\n";
        var_export($data);
    });
   // Abonnement
    $client->subscribe('user-2', function($data){
        echo "user-2\n";
        var_export($data);
    });
    // Zeitgesteuertes Senden von Nachrichten an die Warteschlange
    Timer::add(1, function()use($client){
        $client->send('user-1', ['some', 'data']);
    });
};

Worker::runAll();
```

## API
  * <a href="#construct"><code>Client::<b>__construct()</b></code></a>
  * <a href="#send"><code>Client::<b>send()</b></code></a>
  * <a href="#subscribe"><code>Client::<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>Client::<b>unsubscribe()</b></code></a>

-------------------------------------------------------

<a name="construct"></a>
### __construct (string $address, [array $options])

Instanz erstellen

  * `$address` ähnlich wie `redis://ip:6379`, muss mit redis beginnen.

  * `$options` beinhaltet folgende Optionen:
    * `auth`: Authentifizierungsinformationen, standardmäßig ''
    * `db`: Datenbank, standardmäßig 0
    * `max_attempts`: Anzahl der Wiederholungsversuche nach fehlgeschlagener Verarbeitung, standardmäßig 5
    * `retry_seconds`: Wiederholungsintervall in Sekunden, standardmäßig 5

> Ein fehlgeschlagener Verarbeitungsversuch liegt vor, wenn die Anwendung eine Ausnahme `Exception` oder einen `Error` wirft. Nach einem fehlgeschlagenen Verarbeitungsversuch wird die Nachricht in die verzögerte Warteschlange verschoben und wartet auf einen erneuten Versuch. Die Anzahl der Versuche wird durch `max_attempts` gesteuert, während das Intervall für die Wiederholung von `retry_seconds` und `max_attempts` gemeinsam gesteuert wird. Zum Beispiel, bei einer Einstellung von `max_attempts` auf 5 und `retry_seconds` auf 10, beträgt das Intervall für den ersten Wiederholungsversuch `1*10` Sekunden, für den zweiten Wiederholungsversuch `2*10` Sekunden, und so weiter, bis zum fünften Wiederholungsversuch. Wenn die Anzahl der Wiederholungsversuche die in `max_attempts` festgelegte Anzahl überschreitet, wird die Nachricht in die fehlgeschlagene Warteschlange mit dem Key `{redis-queue}-failed` (vor Version 1.0.5 `{redis-queue-failed}`) verschoben.

-------------------------------------------------------

<a name="send"></a>
### send(String $queue, Mixed $data, [int $dely=0])

Sendet eine Nachricht an die Warteschlange

* `$queue` Warteschlangenname, vom Typ `String`
* `$data` Die zu veröffentlichende Nachricht, kann ein Array oder ein String sein, vom Typ `Mixed`
* `$dely` Verzögerungszeit für die Verarbeitung, standardmäßig 0, vom Typ `Int`
  
-------------------------------------------------------

<a name="subscribe"></a>
### subscribe(mixed $queue, callable $callback)

Abonniert eine oder mehrere Warteschlangen

* `$queue` Warteschlangenname, kann ein String oder ein Array, das mehrere Warteschlangennamen enthält, sein.
* `$callback` Rückruffunktion im Format `function (Mixed $data)`, wobei `$data` die `$data` aus `send($queue, $data)` ist.

-------------------------------------------------------

<a name="unsubscribe"></a>
### unsubscribe(mixed $queue)

Beendet das Abonnement

* `$queue` Warteschlangenname oder ein Array, das mehrere Warteschlangennamen enthält

-------------------------------------------------------

## Senden von Nachrichten an die Warteschlange in einer nicht-Workerman-Umgebung
Manchmal laufen einige Projekte in einer Apache- oder PHP-FPM-Umgebung und können das Workerman/Redis-Queue-Projekt nicht verwenden. In solchen Fällen kann die nachfolgende Funktion verwendet werden, um Nachrichten zu senden.
```
function redis_queue_send($redis, $queue, $data, $delay = 0) {
    $queue_waiting = '{redis-queue}-waiting'; // Vor Version 1.0.5 war es redis-queue-waiting
    $queue_delay = '{redis-queue}-delayed'; // Vor Version 1.0.5 war es redis-queue-delayed
    
    $now = time();
    $package_str = json_encode([
        'id'       => rand(),
        'time'     => $now,
        'delay'    => $delay,
        'attempts' => 0,
        'queue'    => $queue,
        'data'     => $data
    ]);
    if ($delay) {
        return $redis->zAdd($queue_delay, $now + $delay, $package_str);
    }
    return $redis->lPush($queue_waiting.$queue, $package_str);
}
```
Dabei steht der Parameter `$redis` für die Redis-Instanz. Beispielhafte Verwendung mit der Redis-Erweiterung:
```php
$redis = new Redis;
$redis->connect('127.0.0.1', 6379);
$queue = 'user-1';
$data= ['some', 'data'];
redis_queue_send($redis, $queue, $data);
````
