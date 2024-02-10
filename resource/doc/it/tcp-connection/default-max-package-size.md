# maxPackageSize

## Descrizione:
```php
static int Connection::$defaultMaxPackageSize
```

Questa è una proprietà statica globale utilizzata per impostare la lunghezza massima del pacchetto che può essere ricevuto da ogni connessione. Se non impostato, il valore predefinito è 10 MB.

Se la lunghezza del pacchetto ricevuto (valore restituito dal metodo di input della classe del protocollo) è maggiore di ```Connection::$defaultMaxPackageSize```, i dati verranno considerati non validi e la connessione verrà interrotta.


## Esempio

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Impostare la dimensione massima del pacchetto ricevuto per ogni connessione a 1024000 byte
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Avviare il worker
Worker::runAll();
```
