```php
int \ Workerman \ Timer :: add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Führt eine Funktion oder eine Klassenmethode in regelmäßigen Abständen aus.

Beachten Sie: Der Timer wird im aktuellen Prozess ausgeführt. Workerman erstellt keine neuen Prozesse oder Threads, um den Timer auszuführen.

### Parameter
 ``` time_interval ```

Wie lange bis zur nächsten Ausführung? Die Einheit ist die Sekunde und es können Dezimalzahlen verwendet werden, um bis auf 0,001 Sekunden, also bis auf Millisekunden, genau zu sein.

 ``` callback ```

Rückruffunktion. ``` Beachten Sie, dass, wenn die Rückruffunktion eine Methode einer Klasse ist, die Methode öffentlich sein muss. ```

 ``` args ```

Parameter für die Rückruffunktion. Diese müssen als Array übergeben werden.

 ``` persistent ```

Ist der Timer persistent? Wenn die Ausführung nur einmalig erfolgen soll, dann false übergeben (Einmalige Aufgaben werden automatisch nach Abschluss der Ausführung zerstört und es ist nicht notwendig, ```Timer::del()``` aufzurufen). Standardmäßig ist es true, was bedeutet, dass die Ausführung dauerhaft erfolgt.

### Rückgabewert
Gibt eine Ganzzahl zurück, die die Timer-ID repräsentiert. Mit ```Timer::del($timerid)``` kann dieser Timer dann gelöscht werden.

### Beispiel

#### 1. Anonyme Funktion als Timer-Funktion
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// So viele Prozesse wie nötig für die Timer-Aufgaben starten. Beachten, ob das Geschäft bei Mehrprozessen ein Nebenläufigkeitsproblem hat.
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Führen Sie alle 2,5 Sekunden eine Aufgabe aus.
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "Aufgabe läuft\n";
    });
};

// Worker ausführen
Worker::runAll();
```
