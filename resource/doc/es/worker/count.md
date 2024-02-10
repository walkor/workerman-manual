# count

## Descripción:
```php
int Worker::$count
```

Establece cuántos procesos iniciará la instancia actual de Worker, de forma predeterminada es 1 si no se especifica. 

Consulte [**aquí**](../faq/processes-count.md) para obtener información sobre cómo establecer el número de procesos.

Nota: Esta propiedad debe establecerse antes de ejecutar ```Worker::runAll();``` para que sea efectiva. Esta característica no es compatible con el sistema operativo Windows.

## Ejemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Inicia 8 procesos y escucha el puerto 8484 simultáneamente, ofreciendo el servicio mediante el protocolo websocket
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Comenzando el trabajador...\n";
};
// Ejecuta el trabajador
Worker::runAll();
```
