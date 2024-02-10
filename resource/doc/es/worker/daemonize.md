# Daemonize
## Descripción:
```php
static bool Worker::$daemonize
```

Este atributo es un atributo estático global que indica si se está ejecutando en modo daemon (proceso en segundo plano). Si el comando de inicio utiliza el parámetro ```-d```, este atributo se establecerá automáticamente en true. También se puede configurar manualmente en el código.

Nota: Este atributo debe configurarse antes de que se ejecute ```Worker::runAll();```. Este feature no es compatible con el sistema Windows.

## Ejemplo:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Inicio del worker\n";
};
// Ejecutar el worker
Worker::runAll();
```
