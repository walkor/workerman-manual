```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inizializzare un contenitore worker, in ascolto sulla porta 1234
$worker = new Worker('websocket://workerman.net:1234');
// ==== Il numero di processi deve essere impostato su 1 ====
$worker->count = 1;
// Aggiungi un attributo per salvare la mappatura uid su connessione (uid è l'ID utente o l'identificatore univoco del client)
$worker->uidConnections = array();
// Callback quando un client invia un messaggio
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Verifica se il client corrente è già stato autenticato, cioè se è stato impostato un uid
    if(!isset($connection->uid))
    {
       // Se non autenticato, il primo pacchetto viene considerato come uid (qui per comodità di dimostrazione, non viene effettuata una vera e propria autenticazione)
       $connection->uid = $data;
       /* Salva la mappatura uid su connessione, in modo da poter trovare facilmente la connessione tramite uid,
        * per inviare dati specifici per uid
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('accesso riuscito, il tuo uid è ' . $connection->uid);
    }
    // Altre logiche, invio a un uid specifico o broadcast globale
    // Messaggio formato uid:messaggio per inviare a uid
    // uid è 'all' per broadcast globale
    list($recv_uid, $message) = explode(':', $data);
    // Broadcast globale
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // Invia a uid specifico
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// Quando un client si disconnette
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Rimuovi la mappatura alla disconnessione
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

// Invia dati all'uid
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// Esegui tutti i worker (in realtà è stato definito solo uno)
Worker::runAll();
```
