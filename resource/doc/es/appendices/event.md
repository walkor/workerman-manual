# Eventos compatibles actualmente admitidos por Workerman

| Nombre  | Extensiones Dependientes | Admite la programación de subrutinas | Prioridad | Versión de Workerman |
|-----|------|--|-----|
|  Workerman\Events\Select   |   Ninguna   | No admitida  |  Compatible con el núcleo por defecto   |  >=3.0  |
|  Workerman\Events\Revolt   |   event(opcional)   | Sí |  Requiere la instalación de [revolt/event-loop](https://github.com/revoltphp/event-loop)   |  >=5.0  |
|  Workerman\Events\Event   |   event   | No admitida |  Compatible con el núcleo por defecto   |  >=3.0  |
|  Workerman\Events\Swoole   |  [swoole](https://github.com/swoole/swoole-src)   | Sí |  Requiere configuración manual   |  >=4.0  |
|  Workerman\Events\Swow   |   [swow](https://github.com/swow/swow)   | Sí |  Requiere configuración manual   |  >=5.0  |

* Cada controlador de eventos proporcionará características únicas. Por ejemplo, el uso de `Revolt` permitirá que Workerman admita las [subrutinas Fiber](https://www.php.net/manual/zh/language.fibers.php) integradas en PHP, mientras que el uso de `Swoole` permitirá que Workerman admita las subrutinas de Swoole.
* Los controladores de eventos son mutuamente excluyentes. Por ejemplo, al utilizar las subrutinas Fiber de `Revolt`, no se pueden utilizar las subrutinas de Swoole o Swow.
* `Revolt` requiere la instalación de `composer require revolt/event-loop ^1.0.0`. Una vez instalado, el núcleo de Workerman lo configurará automáticamente como el controlador de eventos preferido.
* `Swoole` y `Swow` deben configurarse manualmente con `Worker::$eventLoopClass` para que surtan efecto (consulte el siguiente párrafo).
* Por defecto, Swoole no tiene habilitada la [Runtime de subrutinas](https://wiki.swoole.com/#/runtime?id=runtime), lo que significa que las llamadas basadas en Pdo, Redis y lectura/escritura de archivos integrados en PHP siguen siendo bloqueantes.
* Si desea habilitar el Runtime de subrutinas en Swoole, debe llamar manualmente a `\Swoole\Runtime::enableCoroutine(SWOOLE_HOOK_ALL);`

> **Nota**
> La extensión de swow cambiará automáticamente el comportamiento de algunas funciones integradas en PHP. Esto puede hacer que Workerman no responda a las solicitudes ni a las señales si se ha habilitado la extensión de swow pero no se está utilizando como controlador de eventos. Por lo tanto, si no está utilizando swow como controlador de eventos subyacente, asegúrese de comentar swow en php.ini.

Para obtener más información, consulte [subrutinas de Workerman](../fiber.md).

# Configuración del controlador de eventos para Workerman

A continuación se muestra cómo configurar manualmente el controlador de eventos para Workerman:

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
use Swoole\Coroutine\Http\Client;
use Swoole\Coroutine;

// Configuración manual del controlador de eventos subyacente
Worker::$eventLoopClass = Workerman\Events\Revolt::class;
//Worker::$eventLoopClass = Workerman\Events\Select::class;
//Worker::$eventLoopClass = Workerman\Events\Event::class;
//Worker::$eventLoopClass = Workerman\Events\Swoole::class;
//Worker::$eventLoopClass = Workerman\Events\Swow::class;
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
