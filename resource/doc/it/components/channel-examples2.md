```php
# Esempio 1
**``` (Richiede Workerman versione >= 3.3.0) ```**

Sistema di invio multiprocessuale basato su gruppi di Workers

```php
    use Workerman\Worker;
    use Workerman\Connection\TcpConnection;
    require_once __DIR__ . '/vendor/autoload.php';

    $channel_server = new Channel\Server('0.0.0.0', 2206);

    $worker = new Worker('websocket://0.0.0.0:1234');
    $worker->count = 8;
    // Mappatura globale dei gruppi alle connessioni
    $group_con_map = array();
    $worker->onWorkerStart = function(){
        // Il client Channel si connette al server Channel
        Channel\Client::connect('127.0.0.1', 2206);

        // Ascolta l'evento di invio messaggio del gruppo globale
        Channel\Client::on('send_to_group', function($event_data){
            $group_id = $event_data['group_id'];
            $message = $event_data['message'];
            global $group_con_map;
            var_dump(array_keys($group_con_map));
            if (isset($group_con_map[$group_id])) {
                foreach ($group_con_map[$group_id] as $con) {
                    $con->send($message);
                }
            }
        });
    };
    $worker->onMessage = function(TcpConnection $con, $data){
        // Unisciti al messaggio di gruppo {"cmd":"add_group", "group_id":"123"} 
        // o invia un messaggio di gruppo {"cmd":"send_to_group", "group_id":"123", "message":"Questo è un messaggio"}
        $data = json_decode($data, true);
        var_dump($data);
        $cmd = $data['cmd'];
        $group_id = $data['group_id'];
        switch($cmd) {
            // Connessione al gruppo
            case "add_group":
                global $group_con_map;
                // Aggiungi la connessione all'array di gruppi corrispondente
                $group_con_map[$group_id][$con->id] = $con;
                // Registra a quale gruppo è stata aggiunta questa connessione, utile per pulire i dati di group_con_map corrispondenti al gruppo in onclose
                $con->group_id = isset($con->group_id) ? $con->group_id : array();
                $con->group_id[$group_id] = $group_id;
                break;
            // Invia un messaggio al gruppo
            case "send_to_group":
                // Channel\Client invia l'evento di invio messaggio al gruppo a tutti i processi del server
                Channel\Client::publish('send_to_group', array(
                    'group_id'=>$group_id,
                    'message'=>$data['message']
                ));
                break;
        }
    };
    // È molto importante, quando la connessione viene chiusa, eliminare la connessione dai dati globali del gruppo per evitare memory leak
    $worker->onClose = function(TcpConnection $con){
        global $group_con_map;
        // Scorrere tutti i gruppi a cui è stata aggiunta la connessione e rimuovere i dati corrispondenti da group_con_map
        if (isset($con->group_id)) {
            foreach ($con->group_id as $group_id) {
                unset($group_con_map[$group_id][$con->id]);
                if (empty($group_con_map[$group_id])) {
                    unset($group_con_map[$group_id]);
                }
            }
        }
    };

    Worker::runAll();
```


## Test (assumendo che tutto sia in esecuzione su localhost 127.0.0.1)
1. Eseguire il server
```sh
php start.php start
Workerman[del.php] start in DEBUG mode
----------------------- WORKERMAN -----------------------------
Workerman version:3.4.2          Versione PHP:7.1.3
------------------------ WORKERS -------------------------------
user          worker         listen                    processes status
liliang       ChannelServer  frame://0.0.0.0:2206       1         [OK] 
liliang       none           websocket://0.0.0.0:1234   12        [OK] 
----------------------------------------------------------------
Premere Ctrl-C per uscire. Avvio riuscito.

```

2. Connettere il client al server

Aprire il browser Chrome, premere F12 per aprire la console di debug, e nel pannello Console inserire (o inserire il seguente codice nella pagina html e eseguire in js)

```javascript
// Assumendo che l'IP del server sia 127.0.0.1, modificare l'IP effettivo del server durante il test
ws = new WebSocket('ws://127.0.0.1:1234');
ws.onmessage = function(data){console.log(data.data)};
ws.onopen = function() {
	ws.send('{"cmd":"add_group", "group_id":"123"}');
    ws.send('{"cmd":"send_to_group", "group_id":"123", "message":"Questo è un messaggio"}');
};
```
