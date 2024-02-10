# protocol
Requisiti ```(workerman >= 3.2.7)```

## Descrizione:
```php
string Worker::$protocol
```

Imposta la classe protocollo per l'istanza corrente del Worker.

Nota: la classe di gestione del protocollo può essere specificata direttamente durante l'inizializzazione del Worker nei parametri di ascolto. Ad esempio
```php
$worker = new Worker('http://0.0.0.0:8686');
```

## Esempio


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Avvia il worker
Worker::runAll();
```

Il codice sopra è equivalente al seguente codice


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * Inizialmente verrà controllato se l'utente ha una classe di protocollo personalizzata \Protocols\Http
 * Se non c'è, verrà utilizzata la classe di protocollo integrata di workerman Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Avvia il worker
Worker::runAll();
```
