# sleep
```php
int \Workerman\Timer::sleep(float $delay)
```
Simile alla funzione `sleep()` incorporata in PHP, ma la differenza è che `Timer::sleep()` non è bloccante (non blocca il processo corrente)

> **Nota**
> Questa funzionalità richiede workerman >= 5.0
> Questa funzionalità richiede l'installazione di composer require revolt/event-loop ^1.0.0, oppure l'uso di Swoole/Swow come driver degli eventi


### Parametri
``` delay ```

Intervallo di tempo per eseguire l'azione, espressa in secondi, con supporto per i numeri decimali fino a 0.001, ovvero all'ordine di millisecondi.

### Valore restituito
Nessun valore restituito

### Esempio

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    // Invia i dati dopo un ritardo di 1.5 secondi
    Timer::sleep(1.5);
    // Invia i dati
    $connection->send('hello workerman');
};

Worker::runAll();
```
