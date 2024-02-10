# Latido del corazón

Nota: Las aplicaciones de conexión prolongada deben tener un latido del corazón, de lo contrario, la conexión podría ser desconectada por un nodo de enrutamiento debido a la falta de comunicación durante mucho tiempo.

El latido del corazón tiene principalmente dos funciones:

1. El cliente envía datos al servidor regularmente para evitar que la conexión se cierre debido a la inactividad durante mucho tiempo, lo que puede ser causado por cortafuegos de algunos nodos.

2. El servidor puede determinar si el cliente está en línea a través del latido del corazón. Si el cliente no envía ningún dato dentro del tiempo especificado, se considera que el cliente se desconecta. Esto permite detectar eventos en los que el cliente se desconecta debido a circunstancias extremas como corte de energía, corte de red, entre otros.

Se recomienda el intervalo del latido del corazón:

Se recomienda que el intervalo de envío del latido del corazón por el cliente sea inferior a 60 segundos, por ejemplo, 55 segundos.

> El formato de los datos del latido del corazón no tiene requisitos específicos, siempre que el servidor pueda reconocerlo.

## Ejemplo de latido del corazón
```php
<?php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Intervalo de latido del corazón: 55 segundos
define('HEARTBEAT_TIME', 55);

$worker = new Worker('text://0.0.0.0:1234');

$worker->onMessage = function(TcpConnection $connection, $msg) {
    // Temporalmente se establece un atributo lastMessageTime a la conexión para registrar el tiempo en que se recibió el último mensaje
    $connection->lastMessageTime = time();
    // Otras lógicas de negocio...
};

// Después de que el proceso se inicie, se define un temporizador para ejecutarse cada 10 segundos
$worker->onWorkerStart = function($worker) {
    Timer::add(10, function() use ($worker) {
        $time_now = time();
        foreach($worker->connections as $connection) {
            // Es posible que esta conexión no haya recibido ningún mensaje, por lo que lastMessageTime se establece como el tiempo actual
            if (empty($connection->lastMessageTime)) {
                $connection->lastMessageTime = $time_now;
                continue;
            }
            // Si el intervalo de tiempo desde la última comunicación es mayor que el intervalo de latido del corazón, se considera que el cliente se ha desconectado y se cierra la conexión
            if ($time_now - $connection->lastMessageTime > HEARTBEAT_TIME) {
                $connection->close();
            }
        }
    });
};

Worker::runAll();
```

La configuración anterior indica que si el cliente no envía ningún dato al servidor durante más de 55 segundos, el servidor considerará que el cliente se ha desconectado, cerrará la conexión y activará onClose.

## Re-conexión tras desconexión (Importante)

Tanto si es el cliente el que envía el latido del corazón como si es el servidor quien lo envía, la conexión puede ser desconectada. Por ejemplo, cuando el navegador minimiza la pestaña se suspende el script JS, cambia a otra pestaña, entra en modo de suspensión en la computadora, cambia de red en un dispositivo móvil, la señal está debilitada, la pantalla del teléfono está apagada, la aplicación del teléfono se cambia a segundo plano, problemas en el enrutador, desconexión intencional del negocio, etc. Especialmente en entornos de red externa complejos, muchos nodos de enrutamiento eliminan las conexiones que no están activas durante 1 minuto, por eso se recomienda que el intervalo de latido del corazón sea inferior a 1 minuto.

Las conexiones son fáciles de desconectar en entornos de red externa, por lo que la re-conexión tras desconexión es una función fundamental que debe tener cualquier aplicación de conexión prolongada (la re-conexión tras desconexión solo puede ser realizada por el cliente y no puede ser implementada por el servidor). Por ejemplo, WebSocket del navegador debe escuchar el evento onclose, y cuando se produce onclose, debe establecer una nueva conexión (para evitar bloqueos, se puede retrasar el establecimiento de la conexión). De manera más estricta, el servidor también debería enviar datos de latido del corazón regularmente, y el cliente debe verificar periódicamente si el dato de latido del corazón del servidor no recibe respuesta dentro del tiempo especificado. Si se excede el tiempo especificado sin recibir el dato de latido del corazón del servidor, se considera que la conexión se ha desconectado, se debe cerrar la conexión y establecer una nueva conexión.
