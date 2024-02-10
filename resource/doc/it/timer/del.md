```php
boolean \Workerman\Timer::del(int $timer_id)
```
Elimina un timer specifico

### Parametri
``` timer_id ```

L'ID del timer, cioè un intero restituito dall'interfaccia add

### Valore restituito
boolean


### Esempio
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Avvia n processi per eseguire il compito programmato, attenzione ai problemi di concorrenza multi-processo
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Esegui ogni 2 secondi
    $timer_id = Timer::add(2, function()
    {
        echo "Esecuzione del compito\n";
    });
    // Esegui un compito singolo dopo 20 secondi, eliminando il compito programmato ogni 2 secondi
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Esegui il worker
Worker::runAll();
```

### Esempio (Eliminare il timer corrente nel callback del timer)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Attenzione, l'ID del timer corrente utilizzato nel callback deve essere passato per riferimento (&)
    $timer_id = Timer::add(1, function()use(&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // Dopo 10 esecuzioni, elimina il timer
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Esegui il worker
Worker::runAll();
```
