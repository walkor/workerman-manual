# Come implementare attività asincrone

**Domanda:**

Come gestire in modo asincrono operazioni pesanti per evitare che l'attività principale venga bloccata per un lungo periodo. Ad esempio, devo inviare email a 1000 utenti, un processo lungo che potrebbe bloccare per alcuni secondi. Durante questo processo, poiché il flusso principale viene bloccato, potrebbe influenzare le richieste successive. Come posso delegare questo tipo di attività pesante ad altri processi in modo asincrono?

**Risposta:**

È possibile creare in anticipo alcuni processi di attività su questa macchina o su altri server o addirittura su un cluster di server per gestire le attività pesanti. Il numero di processi di attività può essere aumentato di più, ad esempio 10 volte la CPU, e quindi il chiamante utilizza AsyncTcpConnection per inviare dati in modo asincrono a questi processi di attività per il trattamento asincrono e ottenere i risultati del trattamento asincrono.

Server di processi di attività

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Processo di attività, utilizzando il protocollo di testo
$task_worker = new Worker('Text://0.0.0.0:12345');
// Il numero di processi di attività può essere aumentato in base alle esigenze
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // Supponiamo che i dati inviati siano in formato JSON
     $task_data = json_decode($task_data, true);
     // Gestire la logica dell'attività in base ai dati dell'attività... Ottenere il risultato, qui viene omesso....
     $task_result = ......
     // Invia il risultato
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Chiamata in Workerman

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Servizio websocket
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Stabilisce una connessione asincrona con il servizio di attività remoto, l'IP è l'IP del servizio di attività remoto, se è locale è 127.0.0.1, se è cluster è l'IP di lvs
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Dati dell'attività e dei parametri
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Invia dati
    $task_connection->send(json_encode($task_data));
    // Ottieni il risultato asincrono
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // Risultato
         var_dump($task_result);
         // Dopo aver ottenuto il risultato, ricordati di chiudere la connessione asincrona
         $task_connection->close();
         // Notifica al client websocket corrispondente del completamento dell'attività
         $ws_connection->send('attività completata');
    };
    // Esegue la connessione asincrona
    $task_connection->connect();
};

Worker::runAll();
```

In questo modo, le attività pesanti vengono gestite da processi su questa macchina o su altri server, e una volta completate le attività, verranno ricevuti i risultati in modo asincrono e il processo aziendale non verrà bloccato.
