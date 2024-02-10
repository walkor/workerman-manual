# Cómo implementar tareas asíncronas

**Pregunta:**

¿Cómo manejar de manera asíncrona las operaciones pesadas para evitar que el proceso principal sea bloqueado durante mucho tiempo? Por ejemplo, si quiero enviar correos electrónicos a 1000 usuarios, este proceso es muy lento y puede bloquear la ejecución durante varios segundos, lo cual afectaría las solicitudes posteriores. ¿Cómo puedo delegar este tipo de tareas pesadas a otros procesos para que sean manejadas de manera asíncrona?

**Respuesta:**

Se pueden crear previamente algunos procesos de tarea para manejar operaciones pesadas en la misma máquina, en otros servidores o incluso en un clúster de servidores. El número de procesos de tarea puede ser mayor, por ejemplo, diez veces el número de CPU. Luego, el llamante puede utilizar AsyncTcpConnection para enviar los datos de manera asíncrona a estos procesos de tarea para su procesamiento asíncrono y obtener los resultados de manera asíncrona.

Servidor de procesos de tarea
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Tarea de trabajo, utilizando el protocolo Text
$task_worker = new Worker('Text://0.0.0.0:12345');
// El número de procesos de tarea puede ser mayor según sea necesario
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // Supongamos que los datos recibidos son JSON
     $task_data = json_decode($task_data, true);
     // Procesar la lógica de la tarea según task_data.... obtener el resultado, se omite aquí....
     $task_result = ......
     // Enviar el resultado
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Llamada en webman
```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Servicio WebSocket
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Establecer una conexión asíncrona con el servidor de tareas remoto, la IP es la del servidor de tareas remoto, si es local, sería 127.0.0.1, si es un clúster, sería la IP de LVS
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Datos de la tarea y sus parámetros
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Enviar los datos
    $task_connection->send(json_encode($task_data));
    // Obtener los resultados de manera asíncrona
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // Resultados
         var_dump($task_result);
         // Después de obtener los resultados, recuerde cerrar la conexión asíncrona
         $task_connection->close();
         // Notificar al cliente WebSocket correspondiente que la tarea se ha completado
         $ws_connection->send('tarea completada');
    };
    // Ejecutar la conexión asíncrona
    $task_connection->connect();
};

Worker::runAll();
```

De esta manera, las tareas pesadas se asignan a procesos locales u otros servidores para su ejecución, y se reciben los resultados de manera asíncrona, evitando así el bloqueo del proceso principal.
