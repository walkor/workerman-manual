# Méthode reConnect

```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (exige Workerman version >= 3.3.5) ```

Reconnexion. Généralement appelé dans le rappel ```onClose```, pour effectuer une reconnexion en cas de déconnexion.

En cas de problème réseau ou de redémarrage du service distant, l'appel de cette méthode permet de se reconnecter.

### Paramètre
 ``` $delay ```

Délai avant d'effectuer la reconnexion. Unité en secondes, accepte les décimales et peut être précis jusqu'au milliseconde.

Si aucun paramètre n'est transmis ou si sa valeur est 0, cela signifie une reconnexion immédiate.

Il est recommandé de passer un paramètre pour retarder la reconnexion afin d'éviter une consommation excessive de CPU local en cas de problème persistant avec le service distant.

### Valeur de retour
Aucune valeur de retour

### Exemple

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
        // Si la connexion est interrompue, alors reconnexion dans 1 seconde
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Remarque**
> Après une reconnexion réussie à l'aide de reConnect, la méthode onConnect de $con sera à nouveau appelée (si elle est définie). Il peut être parfois souhaitable que la méthode onConnect ne soit exécutée qu'une seule fois et ne soit pas réexécutée lors de la reconnexion, comme illustré dans l'exemple suivant :

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
        // Si la connexion est interrompue, alors reconnexion dans 1 seconde
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
