# Función del protocolo de comunicación
Dado que TCP se basa en un flujo, los datos de solicitud enviados por el cliente fluyen hacia el servidor como agua. El servidor debe verificar si los datos están completos, ya que solo puede ser una parte de una solicitud que llega al servidor, e incluso puede ser que varias solicitudes lleguen al servidor juntas. Para determinar si las solicitudes han llegado por completo o separar las solicitudes que están juntas, es necesario establecer un protocolo de comunicación.

## ¿Por qué es necesario establecer un protocolo en WorkerMan?
El desarrollo tradicional de PHP se basa en la web y casi siempre en el protocolo HTTP, y el procesamiento del protocolo HTTP es responsabilidad exclusiva del servidor web, por lo que los desarrolladores no necesitan preocuparse por cuestiones de protocolo. Sin embargo, cuando necesitamos desarrollar sobre protocolos no HTTP, los desarrolladores necesitarán considerar los aspectos del protocolo.

## Protocolos admitidos por WorkerMan
En la actualidad, WorkerMan admite HTTP, websocket, protocolo de texto (consulte el apéndice), protocolo de trama (consulte el apéndice) y ws protocolo (consulte el apéndice). Cuando se necesita comunicación basada en alguno de estos protocolos, se puede utilizar directamente especificándolos al inicializar el Worker, por ejemplo:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// websocket://0.0.0.0:2345 indica que esté escuchando el puerto 2345 utilizando el protocolo websocket
$websocket_worker = new Worker('websocket://0.0.0.0:2345');

// Protocolo de texto
$text_worker = new Worker('text://0.0.0.0:2346');

// Protocolo de trama
$frame_worker = new Worker('frame://0.0.0.0:2347');

// Worker TCP, transferencia directa basada en socket, sin usar ningún protocolo de capa de aplicación
$tcp_worker = new Worker('tcp://0.0.0.0:2348');

// Worker UDP, sin usar ningún protocolo de capa de aplicación
$udp_worker = new Worker('udp://0.0.0.0:2349');

// Worker del dominio unix, sin usar ningún protocolo de capa de aplicación
$unix_worker = new Worker('unix:///tmp/wm.sock');

```

## Uso de protocolos de comunicación personalizados
Cuando los protocolos de comunicación incorporados en WorkerMan no satisfacen las necesidades de desarrollo, los desarrolladores pueden personalizar sus propios protocolos de comunicación. Consulte la siguiente sección para obtener información sobre cómo hacerlo.

**Consejo:**
Workerman tiene un protocolo de texto incorporado, el cual tiene un formato de texto más salto de línea. El desarrollo y la depuración del protocolo de texto son muy simples y puede ser utilizado en la mayoría de los escenarios de protocolo personalizado, además es compatible con la depuración mediante telnet. Si un desarrollador desea desarrollar su propio protocolo de aplicación, puede utilizar directamente el protocolo de texto en lugar de desarrollar uno por separado.

Consulte el apéndice de protocolo de texto para obtener más información.
