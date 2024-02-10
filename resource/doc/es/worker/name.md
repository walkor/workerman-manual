# nombre

## Descripción:
```php
string Worker::$name
```

Establece el nombre de la instancia actual del Worker para facilitar la identificación del proceso al ejecutar el comando status. Si no se establece, el nombre por defecto es 'none'.

## Ejemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Establecer el nombre de la instancia
$worker->name = 'MyWebsocketWorker';
$worker->onWorkerStart = function($worker)
{
    echo "Iniciando el worker...\n";
};
// Ejecutar el worker
Worker::runAll();
```
