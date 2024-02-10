# id
Richiesto ```(workerman >= 3.2.1)```

## Descrizione:
```php
int Worker::$id
```

L'identificativo del processo worker corrente, con un range da ```0``` a ```$worker->count-1```.

Questa proprietà è molto utile per distinguere i processi worker. Ad esempio, se un'istanza di worker ha diversi processi e il programmatore vuole impostare un timer solo in uno di essi, può farlo identificando il numero del processo id, ad esempio impostando un timer solo per il processo con id uguale a 0 (vedi esempio).

## Nota:
Dopo il riavvio del processo, il valore dell'identificativo id rimarrà invariato.

L'assegnazione dell'identificativo id è basata su ogni istanza di worker. Ogni istanza di worker inizierà a numerare i propri processi da 0, quindi ci potrebbero essere dei duplicati tra le istanze di worker, ma non all'interno della stessa istanza di worker. Ad esempio, nel seguente esempio:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// L'istanza di worker 1 ha 4 processi, quindi gli id dei processi saranno rispettivamente 0, 1, 2, 3
$worker1 = new Worker('tcp://0.0.0.0:8585');
// Imposta l'avvio di 4 processi
$worker1->count = 4;
// Stampa l'id del processo corrente dopo l'avvio di ogni processo
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// L'istanza di worker 2 ha 2 processi, quindi gli id dei processi saranno rispettivamente 0, 1
$worker2 = new Worker('tcp://0.0.0.0:8686');
// Imposta l'avvio di 2 processi
$worker2->count = 2;
// Stampa l'id del processo corrente dopo l'avvio di ogni processo
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Avvia il worker
Worker::runAll();
```

L'output sarà simile a:
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Nota: poiché il sistema Windows non supporta l'impostazione del numero di processi count, c'è solo un identificativo 0. Inoltre, in Windows non è possibile avviare due worker in ascolto sullo stesso file, quindi questo esempio non funzionerà su Windows.

## Esempio
Un'istanza di worker ha 4 processi e imposta un timer solo nel processo con id uguale a 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Imposta un timer solo nel processo con id uguale a 0, gli altri processi con id 1, 2, 3 non avranno il timer settato
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 processi worker, il timer è impostato solo nel processo con id 0\n";
        });
    }
};
// Avvia il worker
Worker::runAll();
```
