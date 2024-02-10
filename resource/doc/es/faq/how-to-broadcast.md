# Cómo difundir datos

## Ejemplo (difusión programada)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// En este ejemplo, el número de procesos debe ser 1
$worker->count = 1;
// Al iniciar el proceso, configurar un temporizador para enviar datos a todas las conexiones de cliente de manera programada
$worker->onWorkerStart = function($worker)
{
    // Programar, cada 10 segundos
    Timer::add(10, function()use($worker)
    {
        // Recorrer todas las conexiones de cliente actuales del proceso y enviar la hora actual del servidor
        foreach($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Ejecutar el worker
Worker::runAll();
```

## Ejemplo (chat grupal)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// En este ejemplo, el número de procesos debe ser 1
$worker->count = 1;
// Cuando un cliente envía un mensaje, se difunde a otros usuarios
$worker->onMessage = function(TcpConnection $connection, $message)use($worker)
{
    foreach($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Ejecutar el worker
Worker::runAll();
```

## Explicación:
**Un solo proceso:**
Los ejemplos anteriores solo pueden ser **un solo proceso** (```$worker->count=1```), porque con varios procesos, varios clientes pueden conectarse a diferentes procesos, y las conexiones de clientes entre procesos están aisladas y no pueden comunicarse directamente. Es decir, el proceso A no puede operar directamente en el objeto de conexión del cliente del proceso B para enviar datos. (Para lograr esto, se necesita comunicación entre procesos, como el uso del componente Channel, por ejemplo [Ejemplo - Envío de clúster](../components/channel-examples.md), [Ejemplo - Envío de grupos](../components/channel-examples2.md)).

**Se recomienda usar GatewayWorker**
El marco GatewayWorker, desarrollado sobre la base de Workerman, proporciona un mecanismo de empuje más conveniente, incluyendo multicast, broadcast, etc., y se puede configurar con varios procesos e incluso se puede desplegar en varios servidores. Si necesita enviar datos a los clientes, se recomienda utilizar el marco GatewayWorker.

Dirección del manual de GatewayWorker: https://doc2.workerman.net
