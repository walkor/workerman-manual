# Come impostare Workerman per gestire una certa richiesta e riavviare il processo corrente
Per rendere Workerman più compatto, non fornisce direttamente questa impostazione, ma è possibile implementare questa funzionalità con poche righe di codice.
```php
$worker->onMessage = function($connection, $data) {
    static $request_count;
    // La gestione del business è tralasciata
    if(++$request_count > 10000) {
        // Dopo aver ricevuto 10000 richieste, termina il processo corrente, il processo principale avvierà automaticamente un nuovo processo
        Worker::stopAll();
    }
};
```
