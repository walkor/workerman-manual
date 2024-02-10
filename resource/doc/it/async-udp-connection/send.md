# Metodo di invio
```php
void AsyncUdpConnection::send(string $data)
```
Esegue un'operazione di connessione asincrona. Questo metodo restituirà immediatamente.

### Parametri
``` $data ```
I dati da inviare al server, la dimensione dei dati non può superare i 65507 byte (la dimensione massima di trasmissione di un singolo pacchetto UDP è di 65507 byte), altrimenti l'invio avrà esito negativo.

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
    // 1 secondo dopo l'avvio, avvia un client UDP, si connette alla porta 1234 e invia la stringa "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Ricevi i dati di risposta dal server "hello"
            echo "recv $data\r\n";
            // Chiudi la connessione
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Ricevi i dati inviati dal client AsyncUdpConnection, invia la stringa "hello"
    $connection->send("hello");
};
Worker::runAll();
``` 
Stampa qualcosa di simile quando viene eseguito:
```
recv hello
```
