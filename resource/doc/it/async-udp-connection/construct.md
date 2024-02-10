# Metodo __construct
```php
void AsyncUdpConnection::__construct(string $remote_address)
```
Crea un oggetto di connessione UDP.

AsyncUdpConnection consente a Workerman di agire come client e trasmettere dati UDP a un server remoto.

## Parametri
Parametro: `remote_address`

L'indirizzo di connessione, ad esempio
 `udp://192.168.1.1:1234`
 `frame://192.168.1.1:8080`
 `text://192.168.1.1:8080`



## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1 secondo dopo, avvia un client UDP, si connette alla porta 1234 e invia la stringa "hi"
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // Ricezione dei dati di risposta "hello" dal server
            echo "recv $data\r\n";
            // Chiude la connessione
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // Ricezione dei dati inviati dal client AsyncUdpConnection, restituisce la stringa "hello"
    $connection->send("hello");
};
Worker::runAll();             
```

Dopo l'esecuzione, verrà stampato qualcosa di simile a:
```
recv hello
```
