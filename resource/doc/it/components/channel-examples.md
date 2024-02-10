```php
# Esempio 1
**``` (Richiesto Workerman versione>=3.3.0) ```**

Sistema di push multi-processo (cluster distribuito) basato su Worker, cluster per l'invio di gruppo e broadcasting.

`start_channel.php`
Solo un servizio start_channel può essere distribuito nell'intero sistema. Supponiamo che sia eseguito su 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Inizializza un server di canali
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Sono ammessi più servizi start_ws nel sistema, supponendo che siano in esecuzione su due server, 192.168.1.2 e 192.168.1.3.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// server WebSocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Connettiti al server Channel come client di canale
    Channel\Client::connect('192.168.1.1', 2206);
    // Con il proprio id di processo come nome dell'evento
    $event_name = $worker->id;
    // Iscriviti all'evento worker->id e registra la funzione di gestione dell'evento
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "connessione non esistente\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Iscriviti all'evento di broadcasting
    $event_name = 'broadcast';
    // Dopo aver ricevuto l'evento di broadcasting, invia i dati di broadcasting a tutte le connessioni client all'interno del processo corrente
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "workerID:{$worker->id} connectionID:{$connection->id} connesso\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
Sono ammessi più servizi start_ws nel sistema, supponendo che siano in esecuzione su due server, 192.168.1.4 e 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Per gestire le richieste http, invia dati a client arbitrari, è necessario passare workerID e connectionID
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    // Connettiti al server Channel come client di canale
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Compatibilità con workerman4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // Invia dati a una certa connessione client all'interno di un processo worker
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // Broadcasting globale dei dati
    else
    {
        $event_name = 'broadcast';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Test
1. Esegui i servizi sui vari server
2. Collegati al server dal client

Apri il browser Chrome, premi F12 per aprire la console di debug, inserisci (o inserisci il seguente codice nella pagina html e eseguilo con javascript)

```javascript
// Puoi anche connetterti a ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Messaggio dal server ricevuto: " + e.data);
};
```

3. Esegui il push tramite l'interfaccia http

Visita l'URL  ```http://192.168.1.4:4237/?content={$content}``` oppure  ```http://192.168.1.5:4237/?content={$content}``` per inviare i dati ```$content``` a tutte le connessioni client

Visita l'URL ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` oppure ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` per inviare i dati ```$content``` a una certa connessione client all'interno di un processo worker

Nota: Durante il test, sostituire ```{$worker_id}```, ```{$connection_id}``` e ```{$content}``` con i valori effettivi.
