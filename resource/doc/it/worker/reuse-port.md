# reusePort
> **Nota**
> Richiede workerman >= 3.2.1 PHP >= 7.0, il sistema Windows e Mac OS non supportano questa funzionalità

## Descrizione:

```php
bool Worker::$reusePort
```

Imposta se il worker corrente abilita il riutilizzo della porta di ascolto (opzione SO_REUSEPORT del socket).

Dopo aver abilitato il riutilizzo della porta di ascolto, è consentito a più processi non correlati di ascoltare la stessa porta e il kernel di sistema si occuperà del bilanciamento del carico, decidendo quale processo gestire la connessione del socket, evitando l'effetto dello storming degli elefanti e migliorando le prestazioni delle applicazioni multiprocesso a connessione breve.

**Nota:** Questa funzionalità richiede la versione di PHP >= 7.0

## Esempio 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Avvia il worker
Worker::runAll();
```

## Esempio 2: ascolto multiporta (multi-protocollo) workerman
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Dopo l'avvio di ogni processo, aggiunge un'ascolto al processo corrente
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Più processi ascoltano la stessa porta (il socket di ascolto non eredita dal processo padre)
     * È necessario abilitare il riutilizzo della porta, altrimenti verrà generato un errore di "Address already in use"
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Avvia l'ascolto
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Avvia il worker
Worker::runAll();
```
