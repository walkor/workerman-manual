# conexiones
## Descripción:
```php
array Worker::$connections
```
El formato es
```php
array(id=>connection, id=>connection, ...)
```
Esta propiedad almacena todos los objetos de conexión del **proceso actual**. Donde id es el número de identificación de la conexión, consulte el manual [Propiedad id de TcpConnection](../tcp-connection/id.md) para más detalles.

```$connections``` es muy útil para la difusión o para obtener objetos de conexión según el id de la conexión.

Si conoce el número de identificación de la conexión como ```$id```, puede obtener fácilmente el objeto de conexión correspondiente a través de ```$worker->connections[$id]```, y así operar la conexión de socket correspondiente, por ejemplo, enviar datos mediante ```$worker->connections[$id]->send('...')```.

Nota: Si se cierra la conexión (activando onClose), la conexión correspondiente se eliminará del array ```$connections```.

Nota: Los desarrolladores no deben modificar esta propiedad, ya que podría causar situaciones impredecibles.

## Ejemplo
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2020');
// Establece un temporizador al iniciar el proceso para enviar datos a todas las conexiones de clientes periódicamente.
$worker->onWorkerStart = function($worker)
{
    // Cronometrado, cada 10 segundos
    Timer::add(10, function()use($worker)
    {
        // Recorrer todas las conexiones de clientes del proceso actual y enviar la hora del servidor actual
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Ejecuta el worker
Worker::runAll();
```
