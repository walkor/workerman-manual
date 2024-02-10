# Protocolo WebSocket

Actualmente, la versión del protocolo WebSocke en Workerman es **13**.

El protocolo WebSocket es un nuevo protocolo en HTML5 que permite la comunicación full-duplex entre el navegador y el servidor.

## Relación entre WebSocket y TCP

Al igual que HTTP, WebSocket es un protocolo de capa de aplicación que se basa en la transmisión TCP. WebSocket en sí mismo no tiene mucha relación con Socket, y mucho menos son equivalentes.

## Handshake del protocolo WebSocket

El protocolo WebSocket tiene un proceso de handshake, durante el cual el navegador y el servidor comunican a través del protocolo HTTP. En Workerman, es posible intervenir en este proceso de handshake de la siguiente manera:

**Cuando workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // Puede validar aquí si la conexión proviene de manera legítima, de lo contrario se cierra la conexión
        // $_SERVER['HTTP_ORIGIN'] indica desde qué sitio web se originó la conexión websocket
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // Dentro de onWebSocketConnect, $_GET $_SERVER están disponibles
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Cuando workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## Transmisión de datos binarios en el protocolo WebSocket

El protocolo WebSocket por defecto solo puede transmitir texto UTF-8. Para transmitir datos binarios, se debe leer la siguiente sección.

En el protocolo WebSocket, se utiliza un indicador en el encabezado para marcar si se está transmitiendo datos binarios o texto UTF-8. El navegador validará el indicador y el tipo de contenido transmitido. Si no coinciden, se producirá un error y se cerrará la conexión.

Por lo tanto, al enviar datos desde el servidor, es necesario establecer este indicador según el tipo de datos transmitidos. En Workerman, si se trata de texto UTF-8 normal, se debe establecer (generalmente es el valor por defecto, por lo que no es necesario establecerlo manualmente):
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Si se trata de datos binarios, se debe establecer:
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Nota:** Si no se establece `$connection->websocketType`, el valor por defecto será `BINARY_TYPE_BLOB` (es decir, tipo de texto UTF-8). Por lo general, la aplicación transmite texto UTF-8, como datos JSON, por lo que no es necesario establecer manualmente `$connection->websocketType`. Solo se debe establecer este atributo como `BINARY_TYPE_ARRAYBUFFER` al transmitir datos binarios, como datos de imagen o protobuffer, por ejemplo.

## Utilizar workerman como cliente WebSocket

Es posible utilizar la clase [AsyncTcpConnection](../async-tcp-connection.md) junto con el protocolo [ws](about-ws.md) para permitir que workerman actúe como un cliente WebSocket que se conecta a un servidor remoto WebSocket, facilitando la comunicación bidireccional en tiempo real.
