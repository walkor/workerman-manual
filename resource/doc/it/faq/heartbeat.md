# Battito cardiaco

Nota: Le applicazioni a lungo termine devono includere il battito cardiaco, altrimenti la connessione potrebbe essere disconnessa forzatamente dai nodi del router a causa della mancanza di comunicazione per lungo tempo.

Il battito cardiaco ha principalmente due scopi:

1. Il client invia dati al server regolarmente per evitare che la connessione venga interrotta da firewall di alcuni nodi a causa dell'assenza di comunicazioni prolungate.

2. Il server può verificare la presenza del client attraverso il battito cardiaco. Se il client non invia alcun dato entro il tempo prestabilito, verrà considerato offline. Questo consente di individuare eventi in cui il client si disconnette a causa di circostanze estreme (interruzione di corrente, disconnessione dalla rete, ecc.).

Intervallo consigliato del battito cardiaco:

Si consiglia che il client invii il battito cardiaco con un intervallo inferiore a 60 secondi, ad esempio 55 secondi.

> Non ci sono requisiti specifici per il formato dei dati del battito cardiaco, purché il server possa riconoscerli.

## Esempio di battito cardiaco
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Intervallo di battito cardiaco: 55 secondi
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Imposta temporaneamente una proprietà lastMessageTime sulla connessione per registrare l'ultimo momento in cui è stato ricevuto un messaggio
    $connection->lastMessageTime = time();
    // Logica di business aggiuntiva...
};

// Allo start del processo, imposta un timer che si attiva ogni 10 secondi
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function() use ($worker) {
        $time_now = time();
        foreach ($worker->connections as $connection) {
            // Potrebbe essere che questa connessione non ha mai ricevuto alcun messaggio, in tal caso lastMessageTime viene impostato sull'istante attuale
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // Se l'intervallo di tempo dall'ultimo scambio è maggiore dell'intervallo del battito cardiaco, si considera che il client si sia disconnesso e si chiude la connessione
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

Con la configurazione sopra, se il client non invia alcun dato al server per più di 55 secondi, il server considera il client disconnesso, chiude la connessione e attiva l'evento onClose.

## Riconnessione dopo disconnessione (importante)

Sia che sia il client a inviare il battito cardiaco che il server, la connessione può essere interrotta. Ad esempio, quando si riduce al minimo una finestra del browser, lo script JavaScript viene sospeso, oppure quando si passa a un'altra scheda del browser, o quando il computer entra in modalità sleep, e così via. Lo stesso vale anche per i dispositivi mobili quando passano da una rete all'altra, quando il segnale si indebolisce, quando lo schermo del telefono si spegne, quando un'applicazione mobile passa in background e in caso di malfunzionamenti del router o di disconnessioni volontarie. In particolare, in ambienti esterni molto complessi, molti nodi di routing eliminano le connessioni non attive entro 1 minuto, motivo per cui si consiglia di mantenere l'intervallo del battito cardiaco inferiore a 1 minuto.

Le connessioni sono molto facilmente interrotte in ambienti esterni, motivo per cui la riconnessione dopo disconnessione è una funzione obbligatoria per le applicazioni a lungo termine a connessione persistente (la riconnessione dopo disconnessione può essere attuata solo dal client, il server non può implementarla). Ad esempio, WebSocket del browser richiede di ascoltare l'evento onclose e di stabilire una nuova connessione quando si verifica l'evento onclose (per evitare incidenti, è consigliabile ritardare l'impostazione della nuova connessione). Inoltre, è consigliabile che il server invii periodicamente dati del battito cardiaco e che il client controlli periodicamente se è scaduto il tempo prestabilito per ricevere i dati del battito cardiaco dal server. Se non si ricevono i dati del battito cardiaco dal server entro il tempo prestabilito, si deve considerare che la connessione è stata interrotta e si deve eseguire la chiusura, quindi stabilire una nuova connessione.
