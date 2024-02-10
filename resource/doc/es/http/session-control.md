# Descripción
Workerman ha mejorado el soporte para el servicio HTTP a partir de la versión 4.x. Se han introducido clases de solicitud, clases de respuesta, clases de sesión y [SSE](SSE.md). Si deseas utilizar el servicio HTTP de Workerman, se recomienda encarecidamente usar la versión 4.x o una versión posterior.

**Ten en cuenta que todo lo siguiente es para la versión 4.x de Workerman y no es compatible con la versión 3.x.**

## Cambiar el motor de almacenamiento de sesión
Workerman proporciona un motor de almacenamiento de sesión de archivos y un motor de almacenamiento de sesión de Redis. Por defecto se utiliza el motor de almacenamiento de archivos. Si deseas cambiar al motor de almacenamiento de Redis, consulta el siguiente código.

```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Protocols\Http\Session\RedisSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

// Configuración de redis
$config = [
    'host'     => '127.0.0.1', // parámetro obligatorio
    'port'     => 6379,        // parámetro obligatorio
    'timeout'  => 2,           // parámetro opcional
    'auth'     => '******',    // parámetro opcional
    'database' => 1,           // parámetro opcional
    'prefix'   => 'session_'   // parámetro opcional
];
// Utiliza el método Session::handlerClass para cambiar la clase controladora subyacente de la sesión
Session::handlerClass(RedisSessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```

## Configurar la ubicación de almacenamiento de sesión
Cuando se utiliza el motor de almacenamiento predeterminado, los datos de sesión se almacenan por defecto en el disco en la ubicación devuelta por `session_save_path()`. Puedes cambiar la ubicación de almacenamiento mediante el siguiente método.

```php
use Workerman\Worker;
use Workerman\Protocols\Http\Session\FileSessionHandler;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Establecer la ubicación del archivo de sesión
FileSessionHandler::sessionSavePath('/tmp/session');

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Ejecutar el worker
Worker::runAll();
```

## Limpieza de archivos de sesión
Al utilizar el motor de almacenamiento de sesión predeterminado, habrá varios archivos de sesión en el disco. Workerman limpiará los archivos de sesión vencidos según las opciones `session.gc_probability`, `session.gc_divisor` y `session.gc_maxlifetime` establecidas en el php.ini. Consulta la documentación sobre estas tres opciones [aquí](https://www.php.net/manual/zh/session.configuration.php#ini.session.gc-probability).

## Cambiar el controlador de almacenamiento
Además de los motores de almacenamiento de archivos y de Redis, Workerman te permite agregar un nuevo motor de almacenamiento de sesión, como el motor de almacenamiento de MangoDB, el motor de almacenamiento de MySQL, etc., utilizando la interfaz estándar de [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php).

**Procedimiento para agregar un nuevo motor de almacenamiento de sesión**
1. Implementar la interfaz [SessionHandlerInterface](https://www.php.net/manual/zh/class.sessionhandlerinterface.php).
2. Utilizar el método `Workerman\Protocols\Http\Session::handlerClass($class_name, $config)` para reemplazar la interfaz del controlador de sesión subyacente.

**Implementar la interfaz SessionHandlerInterface**

El controlador de almacenamiento personalizado debe implementar la interfaz SessionHandlerInterface, que contiene los siguientes métodos:
```php
SessionHandlerInterface {
    /* Métodos */
    abstract public read ( string $session_id ) : string
    abstract public write ( string $session_id , string $session_data ) : bool
    abstract public destroy ( string $session_id ) : bool
    abstract public gc ( int $maxlifetime ) : int
    abstract public close ( void ) : bool
    abstract public open ( string $save_path , string $session_name ) : bool
}
```
**Explicación de SessionHandlerInterface**
 - El método read se utiliza para leer todos los datos de la sesión correspondientes al session_id desde el almacenamiento. No es necesario deserializar los datos, ya que el framework lo hace automáticamente.
 - El método write se utiliza para escribir los datos de la sesión correspondientes al session_id en el almacenamiento. No es necesario serializar los datos, ya que el framework lo hace automáticamente.
 - El método destroy se utiliza para destruir los datos de la sesión correspondientes al session_id.
 - El método gc se utiliza para eliminar los datos de sesión vencidos, el almacenamiento debe eliminar todas las sesiones cuyo tiempo de modificación sea mayor a maxlifetime.
 - close no requiere ninguna operación, solamente debe devolver true.
 - open no requiere ninguna operación, solamente debe devolver true.

**Reemplazar el controlador subyacente**

Una vez implementada la interfaz SessionHandlerInterface, se utiliza el siguiente método para cambiar el controlador subyacente de la sesión.

```php
Workerman\Protocols\Http\Session::handlerClass($class_name, $config);
```
 - $class_name es el nombre de la clase manejadora de sesiones que implementa la interfaz SessionHandlerInterface. Si tiene un espacio de nombres, debe incluirlo.
 - $config son los parámetros del constructor de la clase SessionHandler.

**Implementación específica**

*Toma en cuenta que esta clase MySessionHandler sirve solo como ejemplo para ilustrar el proceso de cambio del controlador de sesión subyacente, y no debe usarse en un entorno de producción.*
```php
<?php
use Workerman\Worker;
use Workerman\Protocols\Http\Session;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

class MySessionHandler implements SessionHandlerInterface
{
    protected static $store = [];
    
    public function __construct($config) {
        // ['host' => 'localhost']
        var_dump($config);
    }
   
    public function open($save_path, $name)
    {
        return true;
    }

    public function read($session_id)
    {
        return isset(static::$store[$session_id]) ? static::$store[$session_id]['content'] : '';
    }

    public function write($session_id, $session_data)
    {
        static::$store[$session_id] = ['content' => $session_data, 'timestamp' => time()];
    }

    public function close()
    {
        return true;
    }

    public function destroy($session_id)
    {
        unset(static::$store[$session_id]);
        return true;
    }

    public function gc($maxlifetime) {
        $time_now = time();
        foreach (static::$store as $session_id => $info) {
            if ($time_now - $info['timestamp'] > $maxlifetime) {
                unset(static::$store[$session_id]);
            }
        }
    }
}

// Supongamos que la nueva clase MySessionHandler necesita algunos parámetros de configuración
$config = ['host' => 'localhost'];
// Utilizar el método Workerman\Protocols\Http\Session::handlerClass($class_name, $config) para cambiar la clase controladora subyacente de la sesión
Session::handlerClass(MySessionHandler::class, $config);

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('somekey', rand());
    $connection->send($session->get('somekey'));
};

Worker::runAll();
```
