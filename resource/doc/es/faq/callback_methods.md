# Diferentes formas de escribir devoluciones de llamada en PHP

En PHP, la forma más conveniente de escribir devoluciones de llamada es a través de funciones anónimas, pero además de las devoluciones de llamada mediante funciones anónimas, hay otras formas de escribir devoluciones de llamada en PHP. A continuación se muestran ejemplos de varias formas de escribir devoluciones de llamada en PHP.

## 1. Devolución de llamada de función anónima
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Devolución de llamada de función anónima
$http_worker->onMessage = function(TcpConnection $connection, Request $data)
{
    // Enviar 'hello world' al navegador
    $connection->send('hello world');
};

Worker::runAll();
```

## 2. Devolución de llamada de función normal
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// Devolución de llamada de función normal
$http_worker->onMessage = 'on_message';

// Función normal
function on_message(TcpConnection $connection, Request $request)
{
    // Enviar 'hello world' al navegador
    $connection->send('hello world');
}

Worker::runAll();
```

## 3. Método de clase como devolución de llamada
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public function __construct(){}
    public function onWorkerStart(Worker $worker){}
    public function onConnect(TcpConnection $connection){}
    public function onMessage(TcpConnection $connection, $message) {}
    public function onClose(TcpConnection $connection){}
    public function onWorkerStop(Worker $worker){}
}
```
Script de inicio start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Cargar MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Crear un objeto
$my_object = new MyClass();

// Llamar a los métodos de la clase
$worker->onWorkerStart = array($my_object, 'onWorkerStart');
$worker->onConnect     = array($my_object, 'onConnect');
$worker->onMessage     = array($my_object, 'onMessage');
$worker->onClose       = array($my_object, 'onClose');
$worker->onWorkerStop  = array($my_object, 'onWorkerStop');

Worker::runAll();
```

Nota: La estructura de código anterior no permite inicializar recursos (conexiones MySQL, conexiones Redis, conexiones Memcache, etc.) en el constructor, ya que ```$my_object = new MyClass();``` se ejecuta en el proceso principal. En el caso de MySQL, por ejemplo, las conexiones inicializadas en el proceso principal serán heredadas por los subprocesos, y cada subproceso podrá operar con esta conexión a la base de datos. Sin embargo, estas conexiones corresponden a la misma conexión en el servidor MySQL, lo que puede provocar errores imprevisibles, como el error ```mysql gone away```.

Si se necesita inicializar recursos en el constructor de la clase, se puede usar la siguiente sintaxis.
MyClass.php
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    protected $db = null;
    public function __construct(){
        // Suponiendo que la clase de conexión a la base de datos es MyDbClass
        $db = new MyDbClass();
    }
    public function onWorkerStart(Worker $worker){ /* ... */ }
    public function onConnect(TcpConnection $connection){ /* ... */ }
    public function onMessage(TcpConnection $connection, $message) { /* ... */ }
    public function onClose(TcpConnection $connection){ /* ... */ }
    public function onWorkerStop(Worker $worker){ /* ... */ }
}
```
Script de inicio start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Inicializar la clase en onWorkerStart
$worker->onWorkerStart = function($worker) {
    // Cargar MyClass
    require_once __DIR__.'/MyClass.php';
    
    // Crear un objeto
    $my_object = new MyClass();

    // Llamar a los métodos de la clase
    $worker->onConnect    = array($my_object, 'onConnect');
    $worker->onMessage    = array($my_object, 'onMessage');
    $worker->onClose      = array($my_object, 'onClose');
    $worker->onWorkerStop = array($my_object, 'onWorkerStop');
};

Worker::runAll();
```

El bloque de código onWorkerStart se ejecuta en el subproceso correspondiente, lo que significa que cada subproceso establece su propia conexión MySQL, evitando así problemas con la conexión compartida. Además, esta técnica también es compatible con la recarga de código de negocio. Dado que MyClass.php se carga en el subproceso, los cambios en el código de negocio surtirán efecto directamente después de recargar.

## 4. Método estático de clase como devolución de llamada
MyClass.php estática
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
class MyClass{
    public static function onWorkerStart(Worker $worker){ /* ... */ }
    public static function onConnect(TcpConnection $connection){ /* ... */ }
    public static function onMessage(TcpConnection $connection, $message) { /* ... */ }
    public static function onClose(TcpConnection $connection){ /* ... */ }
    public static function onWorkerStop(Worker $worker){ /* ... */ }
}
```
Script de inicio start.php
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Cargar MyClass
require_once __DIR__.'/MyClass.php';

$worker = new Worker("websocket://0.0.0.0:2346");

// Llamar a los métodos estáticos de la clase
$worker->onWorkerStart = array('MyClass', 'onWorkerStart');
$worker->onConnect     = array('MyClass', 'onConnect');
$worker->onMessage     = array('MyClass', 'onMessage');
$worker->onClose       = array('MyClass', 'onClose');
$worker->onWorkerStop  = array('MyClass', 'onWorkerStop');

// Si la clase tiene un espacio de nombres, la sintaxis sería así
// $worker->onWorkerStart = array('tus\nombresapce\MyClass', 'onWorkerStart');
// $worker->onConnect     = array('tus\nombresapce\MyClass', 'onConnect');
// $worker->onMessage     = array('tus\nombresapce\MyClass', 'onMessage');
// $worker->onClose       = array('tus\nombresapce\MyClass', 'onClose');
// $worker->onWorkerStop  = array('tus\nombresapce\MyClass', 'onWorkerStop');

Worker::runAll();
```

Nota: Según el mecanismo de ejecución de PHP, si no se utiliza la construcción new, no se llamará al constructor. Además, los métodos de clase estáticos no permiten el uso de ```$this```.
