# Usuario

## Descripción:
```php
string Worker::$user
```

Establece bajo qué usuario se ejecutará la instancia actual del Worker. Esta propiedad solo es efectiva cuando el usuario actual es root. Si no se establece, se ejecutará con el usuario actual por defecto.

Se recomienda configurar `$user` con un usuario de baja  permisos, como www-data, apache, nobody, etc.

Nota: Esta propiedad debe establecerse antes de ejecutar `Worker::runAll();` para que sea efectiva. Este atributo no es compatible con el sistema Windows.

## Ejemplo
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Establecer el usuario de ejecución de la instancia
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Iniciando el worker...\n";
};
// Ejecutar el worker
Worker::runAll();
```
