# Método reConnect
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Requiere Workerman versión >=3.3.5) ```

Reconectar. Generalmente se llama en la devolución de llamada ```onClose``` para implementar la reconexión después de una desconexión.

Si la conexión se interrumpe debido a problemas de red o reinicio del servicio remoto, se puede llamar a este método para reconectar.

### Parámetros
 ``` $delay ```

Tiempo de espera para ejecutar la reconexión. En segundos, admite valores decimales y puede ser preciso hasta milisegundos.

Si no se pasa ningún valor o si es 0, representa una reconexión inmediata.

Es mejor pasar un parámetro para retrasar la ejecución de la reconexión, para evitar un alto consumo de CPU en la máquina local debido a problemas del servicio remoto que no responde.

### Valor de retorno
Ningún valor de retorno

### Ejemplo
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
> Después de una reconexión exitosa, el método onConnect de $con se llamará nuevamente (si está configurado). A veces queremos que el método onConnect se ejecute solo una vez y no en las reconexiones. Consulte el siguiente ejemplo:

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
