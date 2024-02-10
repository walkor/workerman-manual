# Co-rutina Fiber

Workerman comenzó a soportar Fiber Co-rutina desde la versión 5.0.0 ([Fiber Co-rutina](https://www.php.net/manual/zh/language.fibers.php))

> **Nota**
> La característica de Fiber requiere PHP>=8.1 y la instalación de `composer require revolt/event-loop ^1.0.0`

### Introducción

Fiber es una co-rutina (o fibra) interna de PHP que puede interrumpir el código PHP y luego reanudar su ejecución cuando sea necesario. Su mayor utilidad radica en permitir a los desarrolladores escribir código asíncrono no bloqueante de manera síncrona, lo que mejora en gran medida la mantenibilidad del código.

### Ejemplo
A continuación, se muestra un ejemplo que compara la programación de co-rutinas y la programación asincrónica basada en devolución de llamada.
Supongamos que se requiere llamar a una API HTTP y luego retrasar la respuesta durante un segundo, a continuación se muestra el código para ambos enfoques.

**Programación asincrónica con devolución de llamada**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Llama a la API HTTP
    $http->get('http://example.com/', function ($response) use ($connection) {
        // Enviar después de un segundo de retraso
        Timer::add(1, function() use ($connection, $response) {
            // Enviar datos al navegador
            $connection->send((string)$response->getBody());
        }, null, false);
    });
};

Worker::runAll();
```

**Programación con co-rutinas**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Http\Client;

$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    static $http;
    $http = $http ?: new Client();
    // Llama a la API HTTP
    $response = $http->get('http://example.com/');
    // Retraso de 1 segundo
    Timer::sleep(1);
    // Enviar datos
    $connection->send((string)$response->getBody());
};

Worker::runAll();
```

> **Nota**
> El código anterior requiere la instalación de `composer require workerman/http-client ^2.0.0`

Ambos enfoques son ejecuciones asíncronas no bloqueantes y tienen una buena eficiencia de ejecución, pero el enfoque de co-rutinas es más fácil de leer y mantener en comparación con la devolución de llamada asincrónica.


### Consideraciones sobre Fiber
* Las co-rutinas de Fiber no admiten corutinización de extensiones como Pdo, Redis y funciones de bloqueo internas de PHP, lo que significa que el uso de estas extensiones y funciones seguirá siendo bloqueante.
* Los clientes de co-rutinas de Fiber actualmente disponibles incluyen [workerman/http-client](../components/workerman-http-client.md) y [workerman/redis](../components/workerman-redis.md).

# Co-rutina Swoole
Workerman v5 también admite el uso de Swoole como controlador de eventos subyacente


```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Aquí es necesario configurar manualmente Swoole como controlador de eventos subyacente
Worker::$eventLoopClass = Workerman\Events\Swoole::class;
$worker = new Worker('http://0.0.0.0:12345');
$worker->onMessage = static function($connection, $request)
{
    Coroutine::create(function() use ($connection) {
        $cli = new Client('example.com', 80);
        $cli->get('/get');
        $connection->send($cli->body);
    });
};

Worker::runAll();
```
**Consejo**
* Se recomienda utilizar swoole5.0 o una versión posterior
* Usar Swoole como controlador de eventos subyacente permite que Workerman admita las co-rutinas de Swoole
* Al usar Swoole como controlador de eventos subyacente, no es necesario instalar la extensión event
* Por defecto, Swoole no tiene habilitadas las co-rutinas, lo que significa que las llamadas basadas en Pdo, Redis, escritura y lectura de archivos integrados en PHP son bloqueantes
* Para habilitar las co-rutinas, es necesario llamar manualmente a `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

Para obtener más información, consulta el [manual de Swoole](https://wiki.swoole.com/)

Para más información, consulta [Event Driven](appendices/event.md)

# Acerca de las co-rutinas
En primer lugar, no es necesario fanatizarse con las co-rutinas. Cuando las bases de datos, Redis y otros almacenes están en la red interna, en muchas ocasiones las llamadas bloqueantes multiproceso son más rápidas que las co-rutinas. Según los [datos de pruebas de rendimiento de techempower.com de los últimos 3 años](https://www.techempower.com/benchmarks/#section=data-r21&l=zik073-6bj&test=db), se observa que el rendimiento de las llamadas bloqueantes a la base de datos de Workerman supera al grupo de conexiones de base de datos Swoole + co-rutinas, e incluso supera en casi un 100% al rendimiento de los frameworks de co-rutinas de go, como gin y echo.

Workerman ya ha aumentado el rendimiento de las aplicaciones PHP en varias veces, e incluso varias decenas de veces. En la mayoría de los proyectos de Workerman, las co-rutinas pueden no ofrecer un mayor aumento de rendimiento.
Si tu sistema tiene llamadas lentas, como llamadas HTTP externas, considera utilizar co-rutinas para mejorar el rendimiento.
