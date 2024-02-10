# Protocolo  WS

En la actualidad, la versión del protocolo **WS** de Workerman es la 13.

Workerman puede actuar como cliente, iniciando una conexión websocket a un servidor remoto a través del protocolo **WS**, permitiendo así la comunicación bidireccional.

> **Nota**
> El protocolo **WS** solo puede ser utilizado como cliente a través de la conexión **AsyncTcpConnection** y no puede ser usado como protocolo de escucha para el servidor de **websocket**. Es decir, el siguiente código es incorrecto:

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Si se desea usar Workerman como servidor de **websocket**, por favor utilice el [protocolo de WebSocket](about-websocket.md).

**Ejemplo del protocolo WS como cliente de WebSocket:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Cuando el proceso se inicia
$worker->onWorkerStart = function()
{
    // Conexión al servidor remoto de WebSocket a través del protocolo WS
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Envía un ping al servidor cada 55 segundos con un opcode de 0x9 (opcional)
    $ws_connection->websocketPingInterval = 55;
    // Establece las cabeceras HTTP (opcional)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtraClave' => 'valores'
    ];
    // Establece el tipo de datos (opcional)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB para texto, BINARY_TYPE_ARRAYBUFFER para datos binarios
    // Después de completarse el handshaking TCP (opcional)
    $ws_connection->onConnect = function($connection){
        echo "TCP conectado\n";
    };
    // Después de completarse el handshaking de WebSocket (opcional)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hola');
    };
    // Cuando se recibe un mensaje del servidor remoto de WebSocket
    $ws_connection->onMessage = function($connection, $data){
        echo "recibido: $data\n";
    };
    // Se ejecuta cuando ocurre un error en la conexión, generalmente es un error al conectar con el servidor remoto de WebSocket (opcional)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Cuando la conexión con el servidor remoto de WebSocket se cierra (opcional, se recomienda agregar reconexión)
    $ws_connection->onClose = function($connection){
        echo "conexión cerrada, intentando reconectar\n";
        // Reconectar luego de 1 segundo si la conexión se cerró
        $connection->reConnect(1);
    };
    // Después de configurar todas las devoluciones de llamada anteriores, se ejecuta la operación de conexión
    $ws_connection->connect();
};
Worker::runAll();
```

Para más detalles, verifica [Actuar como cliente de ws/wss](../faq/as-wss-client.md).
