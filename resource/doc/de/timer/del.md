```php
boolean \Workerman\Timer::del(int $timer_id)
```
Löscht einen bestimmten Timer.

### Parameter
``` timer_id ```

Die ID des Timers, die von der add() Methode als Ganzzahl zurückgegeben wird.

### Rückgabewert
boolean

### Beispiel
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Starten von mehreren Prozessen zur Ausführung von Zeitplanaufgaben. Beachten Sie Probleme mit Mehrprozess-Konkurrenz.
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Alle 2 Sekunden ausführen
    $timer_id = Timer::add(2, function()
    {
        echo "task run\n";
    });
    // Führt eine einmalige Aufgabe nach 20 Sekunden aus und löscht die alle 2 Sekunden ausgeführte Zeitplanaufgabe
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Worker ausführen
Worker::runAll();
```

### Beispiel (Löschen des aktuellen Timers im Rückruf des Timers)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Beachten Sie, dass die ID des aktuellen Timers im Rückruf mit dem Referenzzeichen (&) eingeführt werden muss
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // Timer nach 10 Durchläufen löschen
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Worker ausführen
Worker::runAll();
```
