# stopAll
```php
void Worker::stopAll(void)
```

Interrompe il processo corrente e esce.

> **Nota**
> `Worker::stopAll()` viene utilizzato per interrompere il processo corrente. Dopo l'uscita del processo corrente, il processo principale avvierà immediatamente un nuovo processo. Se si desidera interrompere l'intero servizio workerman, si prega di chiamare `posix_kill(posix_getppid(), SIGINT)`

### Parametri
Nessun parametro

### Valore restituito
Nessun valore restituito

## Esempio max_request

Nell'esempio seguente, il sottoprocesso esegue `stopAll` dopo aver gestito 1000 richieste, in modo da riavviare un nuovo processo completamente nuovo. Questo è simile alla proprietà `max_request` di php-fpm e viene principalmente utilizzato per risolvere i problemi di memory leak causati da bug nel codice di business in PHP.

start.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Massimo 1000 richieste per processo
define('MAX_REQUEST', 1000);

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Numero di richieste già processate
    static $request_count = 0;

    $connection->send('hello http');
    // Se il conteggio delle richieste raggiunge 1000
    if(++$request_count >= MAX_REQUEST)
    {
        /*
         * Termina il processo corrente, il processo principale avvierà immediatamente un nuovo processo completamente nuovo
         * completando così il riavvio del processo
         */
        Worker::stopAll();
    }
};

Worker::runAll();
```
