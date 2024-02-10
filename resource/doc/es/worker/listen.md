# listen
```php
void Worker::listen(void)
```
Se utiliza después de instanciar un Worker para empezar a escuchar.

Este método se utiliza principalmente para crear dinámicamente nuevas instancias de Worker después de que el proceso del Worker se haya iniciado. Esto permite a un mismo proceso escuchar en múltiples puertos y admite múltiples protocolos. Es importante tener en cuenta que al usar este método se agrega simplemente una escucha en el proceso actual, no se crea dinámicamente un nuevo proceso ni se activa el método onWorkerStart.

Por ejemplo, si un Worker HTTP se inicia y luego se instancia un Worker WebSocket, ese proceso podrá ser accesible tanto a través del protocolo HTTP como del protocolo WebSocket. Debido a que el Worker WebSocket y el Worker HTTP están en el mismo proceso, pueden acceder a las variables de memoria compartida y compartir todas las conexiones de sockets. Se puede lograr el efecto de recibir una solicitud HTTP y luego operar un cliente WebSocket para completar el envío de datos al cliente.

**Nota:**

Si la versión de PHP es <= 7.0, no es posible instanciar Workers en varios subprocesos para escuchar en el mismo puerto. Por ejemplo, si el proceso A crea un Worker que escucha en el puerto 2016, entonces el proceso B no podrá crear un Worker que escuche en el puerto 2016, de lo contrario se generará un error ```Address already in use```. El siguiente código no se ejecutará:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
// 4 procesos
$worker->count = 4;
// Después de iniciar cada proceso, se agrega una escucha al Worker actual
$worker->onWorkerStart = function($worker)
{
    /**
     * Al iniciar 4 procesos, se crea un Worker que escucha en el puerto 2016
     * Cuando se ejecuta worker->listen(), se generará un error de Address already in use
     * Si worker->count=1, no se generará un error
     */
    $inner_worker = new Worker('http://0.0.0.0:2016');
    $inner_worker->onMessage = 'on_message';
    // Iniciar la escucha. Esto generará un error de Address already in use
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

Si la versión de PHP es >= 7.0, se puede configurar Worker->reusePort=true, lo que permite a múltiples subprocesos crear Workers que escuchen en el mismo puerto. Vea el siguiente ejemplo:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
// 4 procesos
$worker->count = 4;
// Después de iniciar cada proceso, se agrega una escucha al Worker actual
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    // Habilitar reutilización de puertos para crear Workers que escuchen en el mismo puerto (requiere PHP>=7.0)
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Iniciar la escucha. No generará un error si la escucha se realiza adecuadamente
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

### Ejemplo de servidor PHP que envía mensajes en tiempo real a clientes

**Principio:**

1. Establecer un Worker WebSocket para mantener una conexión de larga duración con el cliente.
2. Dentro del Worker WebSocket, se establece un Worker text.
3. El Worker WebSocket y el Worker text son del mismo proceso, lo que permite compartir fácilmente la conexión con el cliente.
4. Alguno de los sistemas PHP independientes se comunica con el Worker text a través del protocolo text.
5. El Worker text opera la conexión WebSocket para enviar datos.

**Código y pasos:**

push.php

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inicializar un contendor de workers, escuchando en el puerto 1234
$worker = new Worker('websocket://0.0.0.0:1234');

/*
 * Es importante que aquí la cantidad de procesos esté configurada como 1
 */
$worker->count = 1;
// Al iniciar el proceso del worker, crear un worker text para abrir un puerto de comunicación interno
$worker->onWorkerStart = function($worker)
{
    // Abrir un puerto interno para facilitar la comunicación interna del sistema que envía datos, el formato del protocolo Text es texto + un salto de línea
    $inner_text_worker = new Worker('text://0.0.0.0:5678');
    $inner_text_worker->onMessage = function(TcpConnection $connection, $buffer)
    {
        // El formato de la matriz $data, que contiene el uid, indica a qué página de ese uid se enviarán los datos
        $data = json_decode($buffer, true);
        $uid = $data['uid'];
        // A través de Workerman, enviar datos a la página de ese uid
        $ret = sendMessageByUid($uid, $buffer);
        // Devolver el resultado del envío
        $connection->send($ret ? 'ok' : 'fail');
    };
    // ## Comenzar a escuchar ##
    $inner_text_worker->listen();
};
// Agregar una propiedad para guardar la correspondencia de uids y conexiones
$worker->uidConnections = array();
// Cuando un cliente envía un mensaje
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Verificar si el cliente actual está verificado, es decir, si se ha establecido un uid
    if(!isset($connection->uid))
    {
       // Si no está verificado, se utiliza el primer paquete como uid (para facilitar la demostración, no se realiza una verificación real)
       $connection->uid = $data;
       /* Guardar la correspondencia de uid con la conexión, de esta manera se puede buscar fácilmente la conexión a través del uid y enviar datos específicos para ese uid */
       $worker->uidConnections[$connection->uid] = $connection;
       return;
    }
};

// Cuando un cliente se desconecta
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Borrar la correspondencia al desconectarse
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Enviar datos a todos los usuarios verificados
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Enviar datos a un uid específico
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
        return true;
    }
    return false;
}

// Ejecutar todos los workers
Worker::runAll();
```

Iniciar el servidor backend
```php push.php start -d```

Código JS para recibir la notificación en el frontend
```javascript
var ws = new WebSocket('ws://127.0.0.1:1234');
ws.onopen = function(){
    var uid = 'uid1';
    ws.send(uid);
};
ws.onmessage = function(e){
    alert(e.data);
};
```

Código PHP para enviar mensajes de notificación desde el backend
```php
// Establecer una conexión por socket al puerto interno de notificación
$client = stream_socket_client('tcp://127.0.0.1:5678', $errno, $errmsg, 1);
// Datos a enviar, incluyendo el campo uid para indicar a qué uid se enviará el mensaje
$data = array('uid'=>'uid1', 'percent'=>'88%');
// Enviar los datos, tener en cuenta que el puerto 5678 es un puerto de protocolo text, por lo que los datos deben tener un salto de línea al final
fwrite($client, json_encode($data)."\n");
// Leer el resultado del envío
echo fread($client, 8192);
```
