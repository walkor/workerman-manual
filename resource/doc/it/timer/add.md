```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Esegue una funzione o un metodo di classe a intervalli regolari.

Nota: il timer viene eseguito nel processo corrente e non genera nuovi processi o thread per eseguire il timer in Workerman.

### Parametri
 ``` time_interval ```

Intervallo di tempo tra le esecuzioni, in secondi, con supporto per numeri decimali, fino a 0,001, cioè fino a millisecondi.

 ``` callback ```

Funzione di richiamo. Nota: se la funzione di richiamo è un metodo di classe, il metodo deve essere di tipo public.

 ``` args ```

Parametri della funzione di richiamo, deve essere un array di valori dei parametri.

 ``` persistent ```

Indica se il timer è persistente. Se si desidera eseguire il timer solo una volta, passare false (i compiti eseguiti una sola volta si autosmettono dopo l'esecuzione e non è necessario invocare ```Timer::del()```). Il valore predefinito è true, quindi il timer viene eseguito regolarmente.

### Valore restituito
Restituisce un intero che rappresenta l'ID del timer e che può essere utilizzato per cancellare il timer chiamando ```Timer::del($timerid)```.

### Esempi

#### 1. Funzione di timer come funzione anonima (chiusura)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Avvia più processi per eseguire il compito temporizzato, fare attenzione a problemi di concorrenza nei processi multipli
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Esegue una funzione ogni 2.5 secondi
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "attività in esecuzione\n";
    });
};

// Esegue il worker
Worker::runAll();
```

### 2. Imposta il timer solo in un processo specifico

Un'istanza di worker ha 4 processi, imposta il timer solo nel processo con ID 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->count = 4;
$worker->onWorkerStart = function(Worker $worker)
{
    // Imposta il timer solo nel processo con ID 0, gli altri processi (1, 2, 3) non impostano nessun timer
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 processi worker, imposta il timer solo nel processo 0\n";
        });
    }
};
// Esegue il worker
Worker::runAll();
```
