# Come trasmettere (broadcast) i dati

## Esempio (broadcast programmato)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// In questo esempio il numero di processi deve essere 1
$worker->count = 1;
// Al momento dell'avvio del processo, imposta un timer per inviare periodicamente dati a tutti i client connessi
$worker->onWorkerStart = function($worker)
{
    // Ogni 10 secondi invia
    Timer::add(10, function()use($worker)
    {
        // Itera su tutte le connessioni attualmente aperte nel processo e invia l'ora attuale del server
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Avvia il worker
Worker::runAll();
```

## Esempio (Chat di gruppo)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// In questo esempio il numero di processi deve essere 1
$worker->count = 1;
// Quando un cliente invia un messaggio, lo invia a tutti gli altri utenti
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Avvia il worker
Worker::runAll();
```

## Spiegazione:
**Singolo processo:**
Gli esempi sopra funzionano solo con **un solo processo** (```$worker->count=1```), poiché con più processi diversi client potrebbero connettersi a processi diversi e le connessioni tra processi sono isolate, quindi non è possibile comunicare direttamente tra di loro. In altre parole, un processo A non può operare **direttamente** sugli oggetti di connessione dei client del processo B (per ottenere questo, è necessaria la comunicazione tra i processi, ad esempio utilizzando il componente Channel, come ad esempio [Esempio - Invio cluster](../components/channel-examples.md), [Esempio - Invio gruppi](../components/channel-examples2.md)).

**È consigliato utilizzare GatewayWorker**
Il framework GatewayWorker, sviluppato sulla base di Workerman, fornisce un meccanismo di trasmissione più conveniente, inclusa la trasmissione a gruppi, la trasmissione broadcast, ecc. Può essere configurato con più processi e persino con più server. Se è necessario inviare dati ai client, è consigliabile utilizzare il framework GatewayWorker.

Indirizzo del manuale di GatewayWorker https://doc2.workerman.net
