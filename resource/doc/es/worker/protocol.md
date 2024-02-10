# protocol

Requerido```（workerman >= 3.2.7）```

## Descripción:
```php
string Worker::$protocol
```

Establece la clase de protocolo para la instancia actual del Worker.

Nota: La clase de manejo de protocolo se puede especificar directamente al inicializar el Worker en los parámetros de escucha. Por ejemplo:
```php
$worker = new Worker('http://0.0.0.0:8686');
```



## Ejemplo


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```

El código anterior es equivalente al siguiente código


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * Primero se verificará si el usuario tiene una clase de protocolo personalizada \Protocols\Http,
 * si no, se usará la clase de protocolo integrada de workerman Workerman\Protocols\Http
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Ejecutar el worker
Worker::runAll();
```
