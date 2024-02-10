```php
void Connection::close(mixed $data = '')
```

Chiudi in modo sicuro la connessione e attiva il callback ```onClose``` della connessione.

Anche se UDP è senza connessione, l'oggetto AsyncUdpConnection corrispondente è sempre mantenuto in memoria e deve essere chiuso chiamando il metodo close per rilasciare l'oggetto di connessione UDP corrispondente, altrimenti quest'ultimo rimarrà in memoria causando una perdita di memoria.

## Parametri

 ``` $data ```

Parametro opzionale, i dati da inviare (se viene specificato un protocollo, verrà chiamato automaticamente il metodo encode del protocollo per impacchettare i dati di ```$data```), una volta che i dati sono stati inviati, la connessione viene chiusa e successivamente verrà attivato il callback onClose.

La dimensione dei dati non può superare i 65507 byte, altrimenti l'invio fallirà.

### Esempio

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 secondo dopo avvia un client UDP, si connette alla porta 1234 e invia la stringa hi
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // Riceve i dati di ritorno dal server hello
            echo "recv $data\r\n";
            // Chiusura della connessione
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // Ricezione dei dati inviati dal client AsyncUdpConnection, restituendo la stringa hello
    $connection->send("hello");
};
Worker::runAll();              
```

Dopo l'esecuzione, verrà stampato qualcosa di simile a:
```
recv hello
```
