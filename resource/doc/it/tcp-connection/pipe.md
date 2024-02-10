# pipe
## Descrizione:
```php
void Connection::pipe(TcpConnection $target_connection)
```



## Parametri
Importa il flusso di dati della connessione corrente nella connessione di destinazione. Include il controllo del traffico integrato. Questo metodo è molto utile per la creazione di proxy TCP.



## Esempio di proxy TCP

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8483');
$worker->count = 12;

// Dopo la creazione della connessione TCP
$worker->onConnect = function(TcpConnection $connection)
{
    // Crea una connessione asincrona alla porta 80 locale
    $connection_to_80 = new AsyncTcpConnection('tcp://127.0.0.1:80');
    // Imposta il reindirizzamento dei dati della connessione del client attuale alla connessione alla porta 80
    $connection->pipe($connection_to_80);
    // Imposta il reindirizzamento dei dati della connessione alla porta 80 alla connessione del client
    $connection_to_80->pipe($connection);
    // Esegue la connessione asincrona
    $connection_to_80->connect();
};

// Esegue il worker
Worker::runAll();
```
