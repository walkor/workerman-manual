# listen
```php
void Worker::listen(void)
```
Utilizzato per eseguire la messa in ascolto dopo l'istanziazione di Worker.

Questo metodo è principalmente utilizzato per creare dinamicamente una nuova istanza di Worker dopo l'avvio del processo Worker, in modo da poter mettere in ascolto lo stesso processo su più porte supportando vari protocolli. È importante notare che l'utilizzo di questo metodo comporta solo l'aggiunta di un ascolto nel processo corrente e non crea dinamicamente nuovi processi né attiva il metodo onWorkerStart.

Ad esempio, dopo l'avvio di un Worker http, istanziando un nuovo Worker websocket, il processo sarà in grado di gestire sia le richieste http che quelle websocket. Poiché i Worker websocket e http sono nello stesso processo, possono accedere a variabili di memoria condivisa e condividere tutte le connessioni di socket. In questo modo è possibile ricevere una richiesta http e poi interagire con il client websocket per inviare dati al client.

**Nota:**

Se la versione di PHP è <= 7.0, non è possibile istanziare Worker con la stessa porta in più processi figlio. Ad esempio, se il processo A crea un Worker in ascolto sulla porta 2016, allora il processo B non può creare un Worker in ascolto sulla porta 2016 altrimenti verrà visualizzato l'errore "Address already in use". Il seguente codice non funzionerà:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 processi
$worker->count = 4;
// Dopo l'avvio di ogni processo, viene creato un nuovo Worker in ascolto nel processo corrente
$worker->onWorkerStart = function($worker)
{
    /**
     * Durante l'avvio dei 4 processi viene creato un Worker in ascolto sulla porta 2016
     * Quando viene eseguito worker->listen() verrà visualizzato l'errore "Address already in use"
     * Se worker->count=1, non visualizzerà alcun errore
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Esegue l'ascolto. Qui verrà visualizzato l'errore "Address already in use"
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

Se la versione di PHP è >= 7.0, è possibile impostare Worker->reusePort=true. In questo modo è possibile creare più Worker con la stessa porta in differenti processi figlio. Ecco un esempio:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 processi
$worker->count = 4;
// Dopo l'avvio di ogni processo, viene creato un nuovo Worker in ascolto nel processo corrente
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Imposta il riutilizzo della porta, in modo da creare Worker in ascolto sulla stessa porta (richiede PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Esegue l'ascolto. Non ci saranno errori in caso di ascolto regolare
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


### Esempio di backend PHP per l'invio di messaggi in tempo reale ai client

**Principio:**

1. Creare un Worker websocket per mantenere una connessione persistente con il client.

2. All'interno del Worker websocket si crea un Worker text.

3. I Worker websocket e text sono nello stesso processo, quindi è possibile condividere facilmente le connessioni dei client.

4. Un sistema back-end PHP indipendente comunica con il Worker text tramite il protocollo text.

5. Il Worker text gestisce la connessione websocket per inviare i dati.

**Codice e passaggi**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inizializza un contenitore worker, in ascolto sulla porta 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * Nota che il numero di processi deve essere impostato su 1
 */
$worker->count = 1;
// Dopo l'avvio del processo worker, crea un worker text per aprire una porta di comunicazione interna
$worker->onWorkerStart = function($worker)
{
    // Apre una porta interna per facilitare l'invio di dati dal sistema interno, usando un formato di testo seguito da un a-capo
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // Il formato dell'array $data, con uid, rappresenta l'invio di dati a quella specifica uid
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // Tramite workerman, invia dati alla pagina uid
        $ret = sendMessageByUid($uid, $buffer);
        // Restituisce il risultato dell'invio
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Inizia l'ascolto ##
    $inner_text_worker->listen();
};
// Aggiunge un attributo per mappare uid alle connessioni
$worker->uidConnections = array();
// Quando un client invia un messaggio
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Verifica se il client è stato già autenticato, cioè se è stato impostato un uid
    if(!isset($connection->uid))
    {
       // Se non ancora autenticato, il primo pacchetto viene considerato come uid (per comodità di dimostrazione, senza autenticazione reale)
       $connection->uid = $data;
       /* Mappa uid alla connessione, in modo da trovare facilmente la connessione per uid,
        * per inviare dati specifici a tale uid.
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// Quando un client si disconnette
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Rimuove la mappatura dalla disconnessione
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Invia dati a tutti gli utenti autenticati
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Invia dati specifici all'uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Avvia tutti i worker
Worker::runAll();
```

Avvia il servizio backend
```php push.php start -d```

Codice JavaScript per ricevere i messaggi push sul front-end
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

Codice per inviare i messaggi di push dal backend
```php
// Stabilisce una connessione socket alla porta di push interna
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Dati da inviare, con il campo uid per indicare a quale uid inviare i dati
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Invia i dati, nota che la porta 5678 è la porta del protocollo di testo, e il protocollo di testo richiede di aggiungere un a-capo alla fine dei dati
fwrite($client, json_encode($data)."\n");
// Legge il risultato dell'invio
echo fread($client, 8192);
```
