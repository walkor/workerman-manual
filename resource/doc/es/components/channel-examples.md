# Example 1
**``` (Requires Workerman version >= 3.3.0) ```**

Sistema de distribución (cluster) de mensajes basado en Worker con múltiples procesos, distribución de mensajes en cluster y difusión en cluster.

`start_channel.php`
Solo se puede desplegar un servicio start_channel en todo el sistema. Supongamos que se ejecuta en 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Inicializar un servidor Channel
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Se pueden desplegar varios servicios start_ws en todo el sistema, suponiendo que se ejecutan en dos servidores: 192.168.1.2 y 192.168.1.3.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Servidor websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Conectar el cliente Channel al servidor Channel
    Channel\Client::connect('192.168.1.1', 2206);
    // Utilizar el ID de proceso como nombre de evento
    $event_name = $worker->id;
    // Suscribirse al evento worker->id y registrar la función manejadora de eventos
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "conexión no existe\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Suscribirse al evento de difusión
    $event_name = 'broadcast';
    // Al recibir el evento de difusión, enviar datos de difusión a todas las conexiones de clientes dentro del proceso actual
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "ID de proceso: {$worker->id} ID de conexión: {$connection->id} conectada\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
Se pueden desplegar varios servicios start_ws en todo el sistema, suponiendo que se ejecutan en dos servidores: 192.168.1.4 y 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Utilizado para manejar peticiones HTTP, difundir datos a cualquier cliente, y se requiere pasar el ID de proceso (workerID) y el ID de conexión (connectionID)
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Compatible con workerman 4.x
    if (!is_array($request)) {
        $_GET = $request->get();
    }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // Enviar datos a una conexión de cliente dentro de un proceso worker específico
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
            'to_connection_id' => $to_connection_id,
            'content'          => $content
        ));
    }
    // Difundir datos globalmente
    else
    {
        $event_name = 'broadcast';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
            'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Prueba

1. Ejecutar los servicios en los servidores respectivos.

2. Conectar el cliente al servidor.

Abrir el navegador Chrome, presionar F12 para abrir la consola de depuración, y en la pestaña de Consola, ingresar (o colocar el siguiente código en una página HTML y ejecutarlo con JavaScript).

```javascript
// También se puede conectar a ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Mensaje recibido del servidor: " + e.data);
};
```

3. Enviar mensajes a través de la interfaz HTTP

Visit the url ```http://192.168.1.4:4237/?content={$content}```  or  ```http://192.168.1.5:4237/?content={$content}``` para enviar el dato ```$content``` a todas las conexiones de clientes.

Visit the url ```http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` or ```http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}``` para enviar el dato ```$content``` a una conexión de cliente específica dentro de un proceso worker.

Nota: Al realizar pruebas, reemplace ```{$worker_id}```, ```{$connection_id}```, y ```{$content}``` con los valores reales.
