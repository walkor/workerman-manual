# Cómo enviar datos a un cliente específico en Workerman
Si estás utilizando Worker para el servidor en lugar de GatewayWorker, ¿cómo puedes enviar mensajes a usuarios específicos?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inicializa un contenedor de workers que escucha en el puerto 1234
$worker = new Worker('websocket://workerman.net:1234');
// ====¡El número de procesos aquí DEBE estar configurado como 1!====
$worker->count = 1;
// Agrega una propiedad para guardar la conexión de uid a conexión (uid es la identificación única del usuario o del cliente)
$worker->uidConnections = array();
// Cuando un cliente envía un mensaje, ejecuta la función de devolución de llamada
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Comprueba si el cliente actual ya está verificado, es decir, si se estableció un uid
    if(!isset($connection->uid))
    {
       // Si no está verificado, el primer paquete se toma como uid (aquí, para mayor comodidad en la demostración, no se realiza una verificación real)
       $connection->uid = $data;
       /* Guarda el uid en la conexión, lo que permite buscar la conexión mediante el uid, 
        * y así enviar datos específicos para dicho uid
        */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('inicio de sesión exitoso, tu uid es ' . $connection->uid);
    }
    // Otra lógica, enviar a un uid específico o hacer una difusión global
    // Se asume que el formato del mensaje es uid:mensaje, para enviar el mensaje al uid
    // Cuando uid es igual a "all", se realiza una difusión global
    list($recv_uid, $message) = explode(':', $data);
    // Difusión global
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // Enviar a un uid específico
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// Cuando un cliente se desconecta
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Al desconectarse, elimina el mapeo
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Envía datos a todos los usuarios verificados
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Envía datos al uid específico
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// Ejecuta todos los workers (en realidad, solo se ha definido uno)
Worker::runAll();
```
**Nota:**
El ejemplo anterior permite la transmisión a un uid específico, aunque sea un solo proceso, soporta hasta 100,000 usuarios en línea sin problemas.


Ten en cuenta que este ejemplo solo admite un solo proceso, es decir, $worker->count DEBE ser 1. Para admitir múltiples procesos o un clúster de servidores, se requiere el componente de Canal para la comunicación entre procesos, el desarrollo también es muy sencillo, puedes consultar la sección [Ejemplo de difusión en clúster con el componente de Canal](../components/channel-examples.md).

**Si deseas enviar mensajes a clientes desde otros sistemas, puedes consultar la sección [Enviar desde otro proyecto](push-in-other-project.md)**
