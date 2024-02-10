```php
# on

**(Richiede Workerman versione >=3.3.0)**

```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```

Sottoscrive all'evento ```$event_name``` e registra la funzione di callback ```$callback_function``` al verificarsi dell'evento.

## Parametri della funzione di callback

 ``` $event_name ```

Nome dell'evento sottoscritto, può essere una stringa qualsiasi.

 ``` $callback_function ```

Funzione di callback scatenata al verificarsi dell'evento. Il prototipo della funzione è ```callback_function(mixed $event_data)```. ```$event_data``` è il dato dell'evento trasmesso al momento della pubblicazione.

Nota:

Se si registra più di una funzione di callback per lo stesso evento, l'ultima funzione di callback registrata sovrascrive la precedente.

## Esempio

Worker multiprocesso (multi-server), un client invia un messaggio, lo trasmette a tutti i clienti.

start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inizializza un server Channel
$channel_server = new Channel\Server('0.0.0.0', 2206);

// Server websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// Ogni volta che un processo worker si avvia
$worker->onWorkerStart = function($worker)
{
    // Il client di Channel si connette al server Channel
    Channel\Client::connect('127.0.0.1', 2206);
    // Sottoscrive all'evento di broadcast e registra la funzione di callback per l'evento
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // Trasmette il messaggio a tutti i client del processo worker attuale
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // Usa i dati inviati dal client come dato dell'evento
   $event_data = $data;
   // Pubblica l'evento di broadcast a tutti i processi worker
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Test**

Apri il browser Chrome, premi F12 per aprire la console di debug, e inserisci il seguente codice nella console (oppure inserisci il codice HTML nella pagina e eseguilo tramite JavaScript).

Connetti il messaggio ricevuto
```javascript
// 127.0.0.1 deve essere sostituito con l'effettivo indirizzo IP di Workerman
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("Messaggio ricevuto dal server: " + e.data);
};
```

Trasmetti il messaggio a tutti
```
ws.send('ciao mondo');
```
