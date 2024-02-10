# Como cliente de ws/wss

A veces es necesario que Workerman actúe como cliente utilizando el protocolo ws/wss para conectarse a un servidor y comunicarse con él. A continuación se muestra un ejemplo.

## Workerman como cliente de ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // Después del éxito del handshake de WebSocket
    $con->onWebSocketConnect = function(AsyncTcpConnection $con, ) {
        $con->send('hello');
    };

    // Cuando se recibe un mensaje
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman como cliente de wss (ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // SSL necesita acceder al puerto 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Establecer el acceso usando SSL para convertirlo en wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman como cliente de wss (ws+ssl) con certificado SSL local

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Configurar la IP local del host remoto y el puerto, así como el certificado SSL
    $context_option = array(
        // Opciones de SSL, consulte http://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Ruta del certificado local. Debe estar en formato PEM e incluir el certificado local y la clave privada.
            'local_cert'        => '/your/path/to/pemfile',
            // Contraseña del archivo local_cert
            'passphrase'        => 'your_pem_passphrase',
            // Permitir certificados autofirmados
            'allow_self_signed' => true,
            // Verificar el certificado SSL
            'verify_peer'       => false
        )
    );

    // SSL necesita acceder al puerto 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Establecer el acceso usando SSL para convertirlo en wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Otras configuraciones

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
    // Conectar al servidor remoto de WebSocket utilizando el protocolo websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Enviar un ping de WebSocket cada 55 segundos al servidor con un opcode de 0x9
    $ws_connection->websocketPingInterval = 55;
    // Encabezados HTTP personalizados
    $ws_connection->headers = ['token' => 'value'];
    // Establecer el tipo de datos, Ws::BINARY_TYPE_BLOB es para texto y Ws::BINARY_TYPE_ARRAYBUFFER es para binario
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB;
    // Después de completar el handshake de TCP
    $ws_connection->onConnect = function($connection){
        echo "TCP conectado\n";
    };
    // Después de completar el handshake de WebSocket
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Cuando se recibe un mensaje del servidor remoto de WebSocket
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // Ocurrió un error, generalmente un error al conectar al servidor remoto de WebSocket
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Cuando se desconecta el servidor remoto de WebSocket
    $ws_connection->onClose = function($connection){
        echo "conexión cerrada e intentando reconectar\n";
        // Si la conexión se desconecta, reconectar en 1 segundo
        $connection->reConnect(1);
    };
    // Después de configurar todos los callbacks, realizar la operación de conexión
    $ws_connection->connect();
};
Worker::runAll();
```
