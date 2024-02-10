# Lectura obligatoria antes del desarrollo

Al desarrollar una aplicación con WorkerMan, es importante comprender los siguientes aspectos:

## 1. Diferencias entre el desarrollo con WorkerMan y el desarrollo PHP convencional

Aunque no hay muchas diferencias significativas entre el desarrollo con WorkerMan y el desarrollo PHP convencional, hay algunas variaciones a tener en cuenta:

### 1.1 Diferencias en el protocolo de la capa de aplicación
* El desarrollo convencional en PHP se basa generalmente en el protocolo de la capa de aplicación HTTP, y el servidor web ya ha completado el análisis del protocolo.
* WorkerMan admite varios protocolos y actualmente tiene integrados los protocolos HTTP y WebSocket. Se recomienda que los desarrolladores utilicen protocolos de comunicación personalizados más simples. Para el desarrollo con el protocolo HTTP, consulte [la sección de servicios HTTP](../http/request.md).

### 1.2 Diferencias en el ciclo de solicitudes
* En una aplicación web en PHP, una vez que se completa una solicitud, se liberan todas las variables y recursos.
* Las aplicaciones desarrolladas con WorkerMan permanecen en la memoria después de la primera carga y análisis, lo que significa que la definición de clases, objetos globales y miembros estáticos de clases no se liberará, lo que facilita su reutilización en el futuro.

### 1.3 Evitar la definición duplicada de clases y constantes
* Debido a que WorkerMan almacena en caché los archivos PHP compilados, es importante evitar cargar en múltiples ocasiones archivos con la definición de clases o constantes. Se recomienda utilizar require_once o include_once para cargar archivos.

### 1.4 Liberación de recursos al utilizar el patrón Singleton
* Dado que WorkerMan no libera objetos globales ni miembros estáticos de clases después de cada solicitud, en patrones singleton relacionados con la base de datos, es común que se guarde la instancia de la base de datos (que incluye una conexión de socket de base de datos) en un miembro estático, lo que permite a WorkerMan reutilizar esta conexión. Sin embargo, es importante tener en cuenta que si el servidor de la base de datos detecta una inactividad prolongada en la conexión, puede cerrar la conexión del socket, lo que provocaría un error al utilizar la instancia de la base de datos nuevamente. WorkerMan proporciona una [clase de base de datos](../components/workerman-mysql.md) con reconexión automática, que los desarrolladores pueden utilizar directamente.

### 1.5 Evitar el uso de declaraciones de salida como exit o die
* WorkerMan se ejecuta en modo de línea de comandos de PHP, por lo que al llamar a declaraciones de salida como exit o die, el proceso actual se detendrá. Aunque el subproceso se reiniciará inmediatamente, esto puede tener un impacto en el negocio.

### 1.6 Reiniciar el servicio después de realizar cambios en el código
Dado que WorkerMan permanece en memoria, una vez que se carga la definición de clases o funciones de PHP, permanece en memoria y no se vuelve a cargar desde el disco. Por lo tanto, después de realizar cambios en el código del negocio, es necesario reiniciar para que los cambios surtan efecto.

## 2. Conceptos básicos a tener en cuenta

### 2.1 Protocolo de transferencia TCP
TCP es un protocolo de transferencia de la capa de transporte basado en IP, orientado a la conexión y confiable. Una característica importante del protocolo TCP es que se basa en un flujo de datos, por lo que las solicitudes de los clientes se envían continuamente al servidor. Por lo tanto, es necesario distinguir los límites de cada solicitud en este flujo continuo de datos. Los protocolos de la capa de aplicación se utilizan principalmente para definir reglas para los límites de las solicitudes y evitar la confusión de datos.

### 2.2 Protocolo de capa de aplicación
Un protocolo de capa de aplicación (protocolo de aplicación) define cómo los procesos de aplicación en diferentes sistemas finales (clientes, servidores) se comunican entre sí. Por ejemplo, HTTP y WebSocket son ejemplos de protocolos de capa de aplicación.

### 2.3 Conexiones de corta duración
Una conexión de corta duración implica que se establece una conexión cuando hay intercambio de datos entre las dos partes, y una vez que se envían los datos, se cierra la conexión. Normalmente, los servicios HTTP de sitios web utilizan conexiones de corta duración.

### 2.4 Conexiones de larga duración
Una conexión de larga duración permite enviar varios paquetes de datos a través de una misma conexión.

### 2.5 Reinicio suave
Mientras que un reinicio convencional implica detener todos los procesos y luego crear nuevos procesos desde cero, durante un reinicio suave, cada proceso se detiene y se reemplaza rápidamente por uno nuevo, hasta que todos los procesos antiguos hayan sido reemplazados.

## 3. Distinguir entre el proceso principal y los subprocesos
Es importante distinguir si el código se está ejecutando en el proceso principal o en un subproceso. En general, el código que se ejecuta antes de la llamada a `Worker::runAll();` se ejecuta en el proceso principal, mientras que el código dentro de las devoluciones de llamada `onXXX` se ejecuta en un subproceso. Es importante destacar que el código que sigue a `Worker::runAll();` nunca se ejecutará.

Por ejemplo, el siguiente código:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Ejecutado en el proceso principal
$tcp_worker = new Worker("tcp://0.0.0.0:2347");
// La asignación se realiza en el proceso principal
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Este fragmento de código se ejecuta en el subproceso
    $connection->send('Hola ' . $data);
};

Worker::runAll();
```

**Nota:** Evite inicializar conexiones de bases de datos, memcache, redis, u otros recursos de conexión en el proceso principal, ya que estas conexiones podrían heredarse automáticamente por los subprocesos (especialmente cuando se usa el patrón Singleton). Todos los subprocesos compartirán la misma conexión, lo que puede provocar confusiones en los datos. Se recomienda inicializar los recursos de conexión en `onWorkerStart`.

