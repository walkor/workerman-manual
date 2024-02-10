# Metodo reConnect
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Richiesto Workerman versione >=3.3.5) ```

Riconnettersi. Di solito chiamato nella funzione di callback ```onClose``` per implementare la riconnessione dopo la disconnessione.

Se la connessione viene interrotta a causa di problemi di rete o riavvio del servizio remoto, è possibile riconnettersi chiamando questo metodo.

### Parametri
``` $delay ```

Indica il ritardo prima di eseguire la riconnessione. L'unità di misura è il secondo, supporta i decimali e può essere preciso fino al millisecondo.

Se non viene passato alcun parametro, o se il valore è 0, significa che la riconnessione avverrà immediatamente.

Si consiglia di passare un parametro per ritardare la riconnessione, in modo da evitare un'elevata cpu locale a causa di problemi di connessione al servizio remoto.

### Valore restituito
Nessun valore restituito

### Esempio

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // If the connection is closed, reconnect after 1 second
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Nota**
> Dopo una riconnessione riuscita, il metodo onConnect di $con verrà chiamato di nuovo (se impostato). Alcune volte si desidera che il metodo onConnect venga eseguito solo una volta e non durante la riconnessione. Si veda l'esempio seguente:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // If the connection is closed, reconnect after 1 second
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
