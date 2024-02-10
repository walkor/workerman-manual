# Descripción

Workerman ha fortalecido el soporte para servicios HTTP desde la versión 4.x. Se han introducido clases de solicitud, clases de respuesta, clases de sesión y [SSE](SSE.md). Si deseas utilizar el servicio HTTP de Workerman, se recomienda encarecidamente utilizar la versión 4.x o posteriores.

**Ten en cuenta que todo lo siguiente corresponde a la versión 4.x de Workerman y no es compatible con la versión 3.x.**

## Notas

- Excepto en el caso de enviar una respuesta por chunk o SSE, no se permite enviar múltiples respuestas en una sola solicitud, es decir, no se permite llamar a `$connection->send()` varias veces en una solicitud.
- Se debe llamar a `$connection->send()` al menos una vez para enviar la respuesta final en cada solicitud, de lo contrario el cliente quedará esperando indefinidamente.

## Respuestas rápidas

Cuando no es necesario cambiar el código de estado HTTP (predeterminado 200) o definir encabezados o cookies personalizados, se puede enviar directamente una cadena al cliente para completar la respuesta.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Envía directamente "this is body" al cliente
    $connection->send("this is body");
};

// Ejecuta el worker
Worker::runAll();
```

## Cambiar código de estado

Cuando es necesario definir un código de estado, encabezados o cookies personalizados, se debe utilizar la clase de respuesta `Workerman\Protocols\Http\Response`. Por ejemplo, el siguiente ejemplo devuelve el estado 404 y el cuerpo del contenido es `<h1>Disculpe, archivo no encontrado</h1>` al acceder a la ruta `/404`.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>Disculpe, archivo no encontrado</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Ejecuta el worker
Worker::runAll();
```
Cuando la clase `Response` ya está inicializada y se quiere cambiar el código de estado, se utiliza el siguiente método.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Enviar encabezados

Del mismo modo, para enviar encabezados se requiere utilizar la clase de respuesta `Workerman\Protocols\Http\Response`.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Valor del encabezado'
    ], 'this is body');
    $connection->send($response);
};

// Ejecuta el worker
Worker::runAll();
```
Una vez que la clase `Response` está inicializada, para añadir o modificar encabezados se utiliza el siguiente método.
```php
$response = new Response(200);
// Añadir o modificar un encabezado
$response->header('Content-Type', 'text/html');
// Añadir o modificar múltiples encabezados
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Valor del encabezado'
]);
$connection->send($response);
```

## Redirección

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## Enviar cookie

Del mismo modo, para enviar cookies se requiere utilizar la clase de respuesta `Workerman\Protocols\Http\Response`.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'this is body');
    $response->cookie('nombre', 'tom');
    $connection->send($response);
};

// Ejecuta el worker
Worker::runAll();
```

## Enviar archivo

Del mismo modo, para enviar archivos se requiere utilizar la clase de respuesta `Workerman\Protocols\Http\Response`.

Cuando se envía un archivo, se realiza de la siguiente manera:
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- Workerman admite el envío de archivos muy grandes.
- Para archivos grandes (más de 2M), Workerman no lee todo el archivo en memoria de una sola vez, sino que lo divide en segmentos y los envía en el momento adecuado.
- Workerman optimiza la velocidad de lectura y envío de archivos según la velocidad de recepción del cliente, garantizando el envío más rápido del archivo con el menor uso de memoria.
- La transferencia de datos es no bloqueante y no afecta el procesamiento de otras solicitudes.
- Al enviar un archivo, automáticamente se agrega el encabezado `Last-Modified`, para que el servidor pueda determinar si enviar una respuesta 304 la próxima vez, lo que ahorra transferencia de archivos y mejora el rendimiento.
- El archivo enviado se envía automáticamente con el encabezado `Content-Type` adecuado para el navegador.
- Si el archivo no existe, se convierte automáticamente en una respuesta 404.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/tu/ruta/del/archivo';
    // Comprobar si el encabezado if-modified-since indica si el archivo ha sido modificado
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // Si el archivo no ha sido modificado, se devuelve un 304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Si el archivo ha sido modificado o no se ha recibido el encabezado if-modified-since, se envía el archivo
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Ejecuta el worker
Worker::runAll();
```

## Enviar datos http chunk

- Debe enviarse primero una respuesta `Response` con el encabezado `Transfer-Encoding: chunked` al cliente.
- Para enviar datos chunk posteriores, se utiliza la clase `Workerman\Protocols\Http\Chunk`.
- Finalmente, se debe enviar un chunk vacío para finalizar la respuesta.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Enviar primero una respuesta con el encabezado Transfer-Encoding: chunked
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Usar la clase Workerman\Protocols\Http\Chunk para enviar datos chunk posteriores
    $connection->send(new Chunk('Primer segmento de datos'));
    $connection->send(new Chunk('Segundo segmento de datos'));
    $connection->send(new Chunk('Tercer segmento de datos'));
   //  Finalmente se envía un chunk vacío para finalizar la respuesta
    $connection->send(new Chunk(''));
};

// Ejecuta el worker
Worker::runAll();
```
