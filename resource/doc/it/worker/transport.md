# trasporto
## Descrizione:
```php
string Worker::$transport
```

Imposta il protocollo di trasporto utilizzato dall'istanza corrente del Worker. Attualmente supporta solo 3 tipi (tcp, udp, ssl). Se non impostato, il default è tcp.

``` Nota: ssl richiede Workerman versione >=3.3.7 ```

## Esempio 1
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Usa il protocollo udp
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Ciao');
};
// Avvia il worker
Worker::runAll();
```

## Esempio 2
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Il certificato dovrebbe essere preferibilmente un certificato valido
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // può anche essere un file .crt
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Qui viene impostato il protocollo websocket
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// Imposta il trasporto attivando l'ssl, quindi websocket+ssl diventa wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
