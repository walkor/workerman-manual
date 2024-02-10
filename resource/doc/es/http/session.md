# Descripción

A partir de la versión 4.x, Workerman ha reforzado el soporte para servicios HTTP. Se han introducido clases de solicitud, clases de respuesta, clases de sesión y [SSE](SSE.md). Si quieres utilizar los servicios HTTP de Workerman, se recomienda encarecidamente que utilices la versión 4.x o una versión posterior.

**Tenga en cuenta que todos los ejemplos siguientes son para la versión 4.x de Workerman y no son compatibles con la versión 3.x.**

# Obtener el objeto de sesión
```php
$session = $request->session();
```

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Ejecutar worker
Worker::runAll();
```

**Consideraciones**
- La sesión debe ser operada antes de llamar a `$connection->send()`.
- La sesión se guarda automáticamente cuando se destruye el objeto, así que no guardes el objeto devuelto por `$request->session()` en un arreglo global o en un miembro de una clase, ya que esto podría impedir que la sesión se guarde.
- La sesión se almacena por defecto en archivos en disco, pero se recomienda usar Redis para obtener un mejor rendimiento.

## Obtener todos los datos de la sesión
```php
$session = $request->session();
$all = $session->all();
```
Devuelve un array. Si no hay datos de sesión, devuelve un array vacío.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// Ejecutar worker
Worker::runAll();
```

## Obtener un valor específico de la sesión
```php
$session = $request->session();
$name = $session->get('name');
```
Devuelve `null` si el dato no existe.

También puedes pasar un valor predeterminado como segundo argumento al método `get`. Si no se encuentra el valor correspondiente en la matriz de sesión, se devolverá el valor predeterminado. Por ejemplo:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
```

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Ejecutar worker
Worker::runAll();
```

## Almacenar datos de sesión
Para almacenar un dato, utiliza el método `set`.
```php
$session = $request->session();
$session->set('name', 'tom');
```
El método `set` no devuelve ningún valor y la sesión se guardará automáticamente al destruir el objeto.

Cuando necesites almacenar múltiples valores, utiliza el método `put`.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
De igual manera, `put` no devuelve ningún valor.

**Ejemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// Ejecutar worker
Worker::runAll();
```
