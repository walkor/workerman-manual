# reusePort
> **Nota**
> Se requiere workerman >= 3.2.1 PHP >= 7.0. Esta característica no es compatible con sistemas Windows y Mac OS.

## Descripción:

```php
bool Worker::$reusePort
```

Establece si el worker actual activa la reutilización del puerto de escucha (opción SO_REUSEPORT del socket).

Después de habilitar la reutilización del puerto de escucha, se permite que múltiples procesos no relacionados escuchen en el mismo puerto, y el equilibrio de carga es manejado por el kernel del sistema para decidir a qué proceso enviar la conexión del socket, evitando el efecto de "temblor en manada" y mejorando el rendimiento de las aplicaciones de múltiples procesos con conexiones cortas.

**Nota:** Esta característica requiere PHP versión >= 7.0

## Ejemplo 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Ejecutar el worker
Worker::runAll();
```

## Ejemplo 2: Escucha múltiple de workerman (múltiple protocolo)

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Cada proceso, después de iniciar, agrega una escucha en el proceso actual
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Múltiples procesos escuchan el mismo puerto (el socket de escucha no hereda del proceso padre)
     * Se requiere habilitar la reutilización del puerto, de lo contrario se producirá un error de "Address already in use"
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Iniciar la escucha
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Ejecutar el worker
Worker::runAll();
```
