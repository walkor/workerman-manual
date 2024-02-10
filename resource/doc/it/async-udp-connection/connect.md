# Metodo connect
```php
void AsyncUdpConnection::connect()
```
Esegue l'operazione di connessione asincrona. Questo metodo ritorna immediatamente.

### Argomenti
Nessun argomento

### Valore restituito
Nessun valore restituito

### Esempio

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';


$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // Avvia un client UDP dopo 1 secondo, si connette alla porta 1234 e invia la stringa "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Riceve i dati di risposta dal server "hello"
            echo "recv $data\r\n";
            // Chiude la connessione
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Riceve i dati inviati dal client AsyncUdpConnection, restituisci la stringa "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

Stamperà qualcosa di simile dopo l'esecuzione:
```
recv hello
```
