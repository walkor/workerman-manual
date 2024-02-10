# ID
Erfordert ```(workerman >= 3.2.1)```

## Beschreibung:
```php
int Worker::$id
```

Die aktuelle ID-Nummer des Worker-Prozesses, die im Bereich von ```0``` bis ```$worker->count-1``` liegt.

Diese Eigenschaft ist sehr nützlich, um Worker-Prozesse zu unterscheiden. Wenn beispielsweise eine Worker-Instanz mehrere Prozesse hat und der Entwickler nur in einem bestimmten Prozess einen Timer setzen möchte, kann dies durch die Erkennung der Prozessnummer ID erreicht werden. Zum Beispiel kann der Timer nur in dem Worker-Prozess mit der ID-Nummer 0 gesetzt werden (siehe Beispiel).

## Hinweis:

Die ID-Nummer bleibt auch nach dem Neustart des Prozesses unverändert.

Die Zuweisung der Prozess-ID basiert auf jeder Worker-Instanz. Jede Worker-Instanz beginnt mit der Zuweisung von ID-Nummern 0 für ihre eigenen Prozesse. Daher können die Prozess-ID-Nummern zwischen den Worker-Instanzen dupliziert werden, aber die ID-Nummern innerhalb einer Worker-Instanz werden sich nicht wiederholen. Wie im folgenden Beispiel:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Worker-Instanz 1 hat 4 Prozesse, deren ID-Nummern jeweils 0, 1, 2, 3 sein werden
$worker1 = new Worker('tcp://0.0.0.0:8585');
// Setze 4 Prozesse in Betrieb
$worker1->count = 4;
// Drucke nach dem Start jedes Prozesses die aktuelle Prozess-ID-Nummer $worker1->id
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// Worker-Instanz 2 hat 2 Prozesse, deren ID-Nummern jeweils 0 und 1 sein werden
$worker2 = new Worker('tcp://0.0.0.0:8686');
// Setze 2 Prozesse in Betrieb
$worker2->count = 2;
// Drucke nach dem Start jedes Prozesses die aktuelle Prozess-ID-Nummer $worker2->id
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Starte die Worker
Worker::runAll();
```
Ausgabe ähnlich wie:
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Hinweis: Da das Windows-Betriebssystem die Einstellung der Prozessanzahl nicht unterstützt, hat die ID-Nummer in Windows nur den Wert 0. Unter Windows wird das Beispiel daher nicht funktionieren, da zwei Worker-Instanzen nicht mit demselben Dateiinitialisierung zwei Listener unterstützen.

## Beispiel
Eine Worker-Instanz hat 4 Prozesse, und es wird nur in dem Prozess mit der ID-Nummer 0 ein Timer festgelegt.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Setze den Timer nur in dem Prozess mit der ID-Nummer 0, die anderen Prozesse mit den ID-Nummern 1, 2, 3 setzen keinen Timer
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 Worker-Prozesse, nur Prozess 0 setzt einen Timer.\n";
        });
    }
};
// Starte die Worker
Worker::runAll();
```
