# Principio de reinicio suave
## ¿Qué es el reinicio suave?

El reinicio suave, a diferencia del reinicio común, permite reiniciar el servicio (generalmente para negocios de conexiones cortas) sin afectar a los usuarios, para volver a cargar el programa PHP y completar la actualización del código empresarial.

El reinicio suave se aplica generalmente durante la actualización comercial o el proceso de publicación de versiones, y evita el impacto temporal de la indisponibilidad del servicio causado por el reinicio del servicio debido a la publicación del código.

> **Nota**
> Windows no admite la recarga.

> **Nota**
> Para negocios de conexión larga (como websockets), las conexiones se interrumpirán durante el reinicio del proceso. La solución es utilizar una arquitectura similar a [gatewayWorker](https://www.workerman.net/doc/gateway-worker), donde un conjunto de procesos mantiene las conexiones y el atributo [reloadable](../worker/reloadable.md) de este conjunto de procesos se establece en falso. El inicio de la lógica comercial se realiza con otro grupo de procesos de trabajadores, y la comunicación entre gateway y los procesos de trabajadores se realiza a través de TCP. Cuando se requiere un cambio comercial, solo es necesario reiniciar los procesos de trabajadores.

## Limitaciones
**Atención: solo los archivos cargados en el callback on{...} se actualizarán automáticamente después del reinicio suave. Las cargas directas de archivos en el script de arranque o el código fijo no se actualizarán automáticamente al reiniciar.**

#### El siguiente código no se actualizará después del reinicio
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function($connection, $request) {
    $connection->send('hi'); // El código fijo no es compatible con la actualización en caliente
};
```

```php
$worker = new Worker('http://0.0.0.0:1234');
require_once __DIR__ . '/your/path/MessageHandler.php'; // Los archivos cargados directamente en el script de inicio no admiten actualizaciones en caliente
$messageHandler = new MessageHandler();
$worker->onMessage = [$messageHandler, 'onMessage']; // Suponiendo que la clase MessageHandler tiene un método onMessage
```


#### El siguiente código se actualizará automáticamente después del reinicio
```php
$worker = new Worker('http://0.0.0.0:1234');
$worker->onWorkerStart = function($worker) { // onWorkerStart es el callback que se activa después de iniciar el proceso
    require_once __DIR__ . '/your/path/MessageHandler.php'; // Los archivos cargados después de iniciar el proceso admiten actualizaciones en caliente
    $messageHandler = new MessageHandler();
    $worker->onMessage = [$messageHandler, 'onMessage'];
};
```
Después de realizar cambios en MessageHandler.php, al ejecutar `php start.php reload`, MessageHandler.php se recargará en memoria para actualizar la lógica comercial.

> **Consejo**
> En el código anterior, para facilitar la demostración, se utilizó la instrucción `require_once`, si su proyecto admite la carga automática de PSR4, no es necesario llamar a la instrucción `require_once`.

## Principio de reinicio suave

WorkerMan se divide en el proceso principal y los subprocesos. El proceso principal monitorea los subprocesos, mientras que los subprocesos se encargan de aceptar las conexiones de los clientes y los datos de solicitud enviados a través de las conexiones, realizan el procesamiento correspondiente y devuelven los datos al cliente. Cuando se actualiza el código empresarial, en realidad solo necesitamos actualizar los subprocesos para lograr el objetivo.

Cuando el proceso principal de WorkerMan recibe la señal de reinicio suave, enviará una señal de apagado seguro (para permitir que el proceso asociado complete la solicitud actual antes de apagarse) a uno de los subprocesos. Después de que este proceso se apague, el proceso principal creará nuevamente un nuevo subproceso (este subproceso cargará el nuevo código PHP), luego el proceso principal enviará nuevamente el comando de detener a otro proceso antiguo, reiniciando así un proceso a la vez hasta que todos los procesos antiguos sean reemplazados.

Vemos que el reinicio suave en realidad implica que los viejos procesos comerciales se apagan uno por uno y se crean new procesos uno por uno. Para no afectar a los clientes durante el reinicio suave, esto requiere que los procesos no mantengan información de estado relacionada con los usuarios, es decir, es preferible que los procesos comerciales no tengan estado para evitar la pérdida de información debido al cierre del proceso.
