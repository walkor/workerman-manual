# Documentación de webman

workerman ha mejorado el soporte del servicio HTTP desde la versión 4.x. Introdujo la clase de solicitud, clase de respuesta, clase de sesión, y [SSE](SSE.md). Si quieres utilizar el servicio HTTP de workerman, se recomienda encarecidamente utilizar la versión 4.x de workerman o una versión superior.

**Tenga en cuenta que todos los siguientes ejemplos son para la versión 4.x de workerman y no son compatibles con la versión 3.x.**

## Obtener el objeto de solicitud
Debes obtener el objeto de solicitud dentro de la función de devolución de llamada `onMessage`. El framework automáticamente pasa el objeto de solicitud como segundo parámetro a la función de devolución de llamada.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request es el objeto de solicitud, aquí no se realizan operaciones en el objeto de solicitud, simplemente se devuelve "hello" al navegador.
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```

Cuando un navegador visita `http://127.0.0.1:8080`, devolverá `hello`.

## Obtener parámetros de solicitud GET

**Obtener todo el array GET**
```php
$get = $request->get();
```
Si la solicitud no tiene parámetros GET, devolverá un array vacío.

**Obtener un valor específico del array GET**
```php
$name = $request->get('name');
```
Si el array GET no contiene este valor, devolverá null.

También puedes proporcionar un valor predeterminado como segundo parámetro para el método `get`. Si el valor correspondiente no se encuentra en el array GET, devolverá el valor predeterminado. Por ejemplo:
```php
$name = $request->get('name', 'tom');
```

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->get('name'));
};

// Ejecutar el worker
Worker::runAll();
```

Cuando un navegador visita `http://127.0.0.1:8080?name=jerry&age=12`, devolverá `jerry`.

## Obtener parámetros de solicitud POST

**Obtener todo el array POST**
```php
$post = $request->post();
```
Si la solicitud no tiene parámetros POST, devolverá un array vacío.

**Obtener un valor específico del array POST**
```php
$name = $request->post('name');
```
Si el array POST no contiene este valor, devolverá null.

Al igual que el método `get`, también puedes proporcionar un valor predeterminado como segundo parámetro para el método `post`. Si el valor correspondiente no se encuentra en el array POST, devolverá el valor predeterminado. Por ejemplo:
```php
$name = $request->post('name', 'tom');
```

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// Ejecutar el worker
Worker::runAll();
```

## Obtener el cuerpo POST original de la solicitud
```php
$post = $request->rawBody();
```
Esta funcionalidad es similar a la operación `file_get_contents("php://input")` en `php-fpm`. Se utiliza para obtener el cuerpo original de la solicitud HTTP, lo cual es útil para obtener datos de solicitud POST en formato no `application/x-www-form-urlencoded`.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// Ejecutar el worker
Worker::runAll();
```

## Obtener el encabezado
**Obtener todo el array de encabezado**
```php
$headers = $request->header();
```
Si la solicitud no tiene encabezados, devolverá un array vacío. Todos los claves están en minúsculas.

**Obtener un valor específico del array de encabezado**
```php
$host = $request->header('host');
```
Si el array de encabezado no contiene este valor, devolverá null. Todos los claves están en minúsculas.

Al igual que los métodos `get` y `post`, también puedes proporcionar un valor predeterminado como segundo parámetro para el método `header`. Si el array de encabezado no contiene el valor correspondiente, devolverá el valor predeterminado. Por ejemplo:
```php
$host = $request->header('host', 'localhost');
```

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// Ejecutar el worker
Worker::runAll();
```

## Obtener cookies
**Obtener todo el array de cookies**
```php
$cookies = $request->cookie();
```
Si la solicitud no tiene cookies, devolverá un array vacío.

**Obtener un valor específico del array de cookies**
```php
$name = $request->cookie('name');
```
Si el array de cookies no contiene este valor, devolverá null.

Al igual que los métodos `get` y `post`, también puedes proporcionar un valor predeterminado como segundo parámetro para el método `cookie`. Si el array de cookies no contiene el valor correspondiente, devolverá el valor predeterminado. Por ejemplo:
```php
$name = $request->cookie('name', 'tom');
```

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// Ejecutar el worker
Worker::runAll();
```

## Obtener archivos subidos
**Obtener todo el array de archivos subidos**
```php
$files = $request->file();
```
El formato del archivo devuelto es similar a:
```php
array (
    'avatar' => array (
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' =>  array (
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
Donde:
 - `name` es el nombre del archivo
 - `tmp_name` es la ubicación del archivo temporal en disco
 - `size` es el tamaño del archivo
 - `error` es el [código de error](https://www.php.net/manual/zh/features.file-upload.errors.php)
 - `type` es el tipo MIME del archivo.

**Nota:**
 - El tamaño de los archivos subidos está limitado por [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md), que es 10M por defecto y puede ser modificado.
 - Después de que se complete la solicitud, los archivos serán eliminados automáticamente.
 - Si la solicitud no tiene archivos subidos, devolverá un array vacío.

### Obtener un archivo subido específico
```php
$avatar_file = $request->file('avatar');
```
Devolverá algo similar a
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
Si el archivo subido no existe, devolverá null.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// Ejecutar el worker
Worker::runAll();
```

## Obtener el host
Obtener la información del host de la solicitud.
```php
$host = $request->host();
```
Si la dirección de la solicitud no es estándar en el puerto 80 o 443, la información del host puede incluir el puerto, por ejemplo, `example.com:8080`. Si no se necesita el puerto, se puede pasar `true` como primer parámetro.

```php
$host = $request->host(true);
```

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->host());
};

// Ejecutar el worker
Worker::runAll();
```

Cuando un navegador visita `http://127.0.0.1:8080?name=tom`, devolverá `127.0.0.1:8080`.
## Obteniendo el método de la solicitud

```php
$method = $request->method();
```
El valor devuelto puede ser uno de `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `HEAD`.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// Ejecutar el worker
Worker::runAll();
```

## Obteniendo el URI de la solicitud
```php
$uri = $request->uri();
```
Devuelve el URI de la solicitud, incluyendo la parte del path y queryString.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// Ejecutar el worker
Worker::runAll();
```
Cuando el navegador accede a `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, devolverá `/user/get.php?uid=10&type=2`.

## Obteniendo el path de la solicitud
```php
$path = $request->path();
```
Devuelve la parte del path de la solicitud.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// Ejecutar el worker
Worker::runAll();
```
Cuando el navegador accede a `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, devolverá `/user/get.php`.

## Obteniendo el queryString de la solicitud
```php
$query_string = $request->queryString();
```
Devuelve la parte del queryString de la solicitud.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// Ejecutar el worker
Worker::runAll();
```
Cuando el navegador accede a `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, devolverá `uid=10&type=2`.

## Obteniendo la versión del protocolo HTTP de la solicitud
```php
$version = $request->protocolVersion();
```
Devuelve la cadena `1.1` o `1.0`.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// Ejecutar el worker
Worker::runAll();
```

## Obteniendo el sessionId de la solicitud
```php
$sid = $request->sessionId();
```
Devuelve una cadena compuesta por letras y números.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// Ejecutar el worker
Worker::runAll();
```
