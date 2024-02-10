# send
## Descripción:
```php
mixed Connection::send(mixed $data [,$raw = false])
```

Envía datos al cliente.

## Parámetros

``` $data ```

Los datos a enviar. Si se especifica un protocolo al inicializar la clase Worker, se llamará automáticamente al método encode del protocolo para empaquetar los datos y enviarlos al cliente.

``` $raw ```

Indica si se deben enviar los datos en bruto, es decir, sin llamar al método encode del protocolo. Por defecto es false, lo que significa que se llamará al método encode del protocolo automáticamente.

## Valor devuelto

true: Indica que los datos se han escrito correctamente en el búfer de envío del socket de esa conexión en el nivel del sistema operativo.

null: Indica que los datos se han escrito en el búfer de envío de la aplicación de esa conexión, a la espera de ser escritos en el búfer de envío del socket del sistema operativo.

false: Indica que ha fallado el envío, debido a que la conexión del cliente ya se ha cerrado o que el búfer de envío de la aplicación de esa conexión está completo.

## Notas
Cuando send devuelve ```true```, solo significa que los datos se han escrito correctamente en el búfer de envío del socket de esa conexión en el nivel del sistema operativo, y no implica que los datos se hayan enviado con éxito al búfer de recepción del socket de destino, ni mucho menos que la aplicación de destino haya leído los datos del búfer de recepción del socket local. **Sin embargo, siempre y cuando send no devuelva false, la red esté activa y el cliente esté recibiendo normalmente, esencialmente se puede considerar que los datos se están enviando al destinatario al 100%.**

Debido a que los datos en el búfer de envío del socket se envían de forma asincrónica al destino por el sistema operativo, el sistema operativo no proporciona un mecanismo de confirmación correspondiente a la aplicación, por lo que **la aplicación** no puede saber cuándo comienzan a enviarse los datos desde el búfer de envío del socket, ni puede saber si los datos del búfer de envío del socket se han enviado con éxito. Por estas razones, Workerman no puede proporcionar directamente una interfaz de confirmación de mensajes.

Si su negocio requiere garantizar que cada mensaje llegue al cliente, puede agregar un mecanismo de confirmación en su negocio. El mecanismo de confirmación puede variar según el negocio, y puede haber varias formas de mecanismo de confirmación incluso para el mismo negocio.

Como ejemplo, un sistema de chat puede utilizar el siguiente mecanismo de confirmación: guardar cada mensaje en la base de datos y cada mensaje tiene un campo de "leído" o "no leído". Cuando un cliente recibe un mensaje, envía un paquete de confirmación al servidor, que marca el mensaje correspondiente como leído. Cuando un cliente se conecta al servidor (generalmente cuando inicia sesión o reconecta después de una desconexión), el servidor verifica si hay mensajes no leídos en la base de datos, y si los hay, los envía al cliente. Del mismo modo, cuando el cliente recibe un mensaje, notifica al servidor que se ha leído. De esta manera, se puede garantizar que el otro cliente reciba todos los mensajes. Por supuesto, los desarrolladores también pueden usar su lógica de confirmación personalizada.

## Ejemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data) {
    // Se llamará automáticamente a \Workerman\Protocols\Websocket::encode para empaquetar los datos en el protocolo websocket antes de enviarlos
    $connection->send("hello\n");
};
// Ejecutar el worker
Worker::runAll();
```
