# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
Ähnlich wie die in PHP eingebaute `sleep()` Funktion, jedoch ist `Timer::sleep()` nicht blockierend (blockiert den aktuellen Prozess nicht).

> **Hinweis**
> Diese Funktion erfordert workerman>=5.0
> Diese Funktion erfordert die Installation von composer require revolt/event-loop ^1.0.0 oder die Verwendung von Swoole/Swow als Event-Driven

### Parameter
 ``` delay ```

Die Zeitdauer, nach der die Funktion ausgeführt werden soll, in Sekunden. Es werden auch Dezimalzahlen unterstützt, genau bis zu 0,001, also auf Millisekundenbasis.

### Rückgabewert
Kein Rückgabewert

### Beispiel

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // Daten werden mit einer Verzögerung von 1,5 Sekunden gesendet
    Timer::sleep(1.5);
    // Daten senden
    $connection->send('hello workerman');
};

Worker::runAll();
```
