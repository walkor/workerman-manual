# Quanti processi dovrebbero essere attivati

## Come impostare il numero dei processi
Il numero dei processi è determinato dall'attributo ```count``` (il sistema Windows non supporta l'impostazione del numero dei processi), ad esempio nel codice seguente:
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## Avvia 4 processi per fornire servizi esterni ##
$http_worker->count = 4;

...
```

## I fattori da considerare per l'impostazione del numero dei processi
1. Numero di core della CPU
2. Quantità di memoria
3. Orientamento del business verso processi IO-intensivi o CPU-intensivi

## Principi per l'impostazione del numero dei processi
1. La somma della memoria occupata da ciascun processo deve essere inferiore alla memoria totale (generalmente, la memoria occupata da ciascun processo aziendale è di circa 40 MB)
2. Se l'attività è di tipo IO-intensivo, ovvero coinvolge I/O bloccanti come l'accesso a Mysql, Redis e altri database, il numero dei processi può essere aumentato, ad esempio, impostandolo a 3 volte il numero di core della CPU. Se vi sono numerose attese bloccanti nell'attività, il numero dei processi può essere aumentato ulteriormente, ad esempio fino a 8 volte il numero dei core della CPU. Si noti che l'I/O non bloccante è di tipo CPU-intensivo, non IO-intensivo.
3. Se l'attività è di tipo CPU-intensivo, ovvero non comporta spese in I/O bloccanti, ad esempio l'uso di lettura asincrona per risorse di rete, il numero dei processi può essere impostato uguale al numero di core della CPU.

## Valori di riferimento per l'impostazione del numero dei processi
Se il codice aziendale è orientato verso l'I/O, il numero dei processi può essere impostato in base al grado di intensità dell'I/O, ad esempio da 3 a 8 volte il numero dei core della CPU.
Se il codice aziendale è orientato verso la CPU, il numero dei processi può essere impostato uguale al numero dei core della CPU.

## Attenzione
L'I/O di WorkerMan è sempre non bloccante, ad esempio, ```Connection->send``` è un'operazione non bloccante, quindi rientra nella categoria di operazioni CPU-intensive. Se non si conosce l'orientamento del proprio business, è sufficiente impostare il numero dei processi a circa 3 volte il numero dei core della CPU.
Inoltre, un numero maggiore di processi non significa necessariamente una migliore prestazione. Un numero eccessivo di processi aumenterà i costi di commutazione dei processi, influenzando le prestazioni.
