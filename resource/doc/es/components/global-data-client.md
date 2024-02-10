# Cliente del componente GlobalData
**```(Requiere Workerman versión >= 3.3.0)```**

# __construct
```php
void \GlobalData\Client::__construct(mixed $server_address)
```

Instancia un objeto cliente \GlobalData\Client. Comparte datos entre procesos asignando propiedades al objeto cliente.

### Parámetros
Dirección del servidor GlobalData, con el formato ```<dirección IP>:<puerto>```, por ejemplo ```127.0.0.1:2207```.

Para un clúster de servidores GlobalData, pasar un array de direcciones, por ejemplo ```array('10.0.0.10:2207', '10.0.0.0.11:2207')```.

## Descripción
Soporta operaciones de asignación, lectura, isset, unset. También soporta operaciones atómicas CAS.

## Ejemplos

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Servidor GlobalData
$global_worker = new GlobalData\Server('0.0.0.0', 2207);

$worker = new Worker('tcp://0.0.0.0:6636');
// Al iniciar el proceso
$worker->onWorkerStart = function()
{
    // Inicializa un cliente global data global
    global $global;
    $global = new \GlobalData\Client('127.0.0.1:2207');
};
// Cada vez que el servidor recibe un mensaje
$worker->onMessage = function(TcpConnection $connection, $data)
{
    // Cambia el valor de $global->somedata, este valor será compartido entre otros procesos
    global $global;
    echo "ahora global->somedata=".var_export($global->somedata, true)."\n";
    echo "asignar \$global->somedata=$data";
    $global->somedata = $data;
};
Worker::runAll();
```

### Todos los usos posibles (también en entorno php-fpm)
```php
require_once __DIR__ . '/vendor/autoload.php';

$global = new Client('127.0.0.1:2207');

var_export(isset($global->abc));

$global->abc = array(1,2,3);

var_export($global->abc);

unset($global->abc);

var_export($global->add('abc', 10));

var_export($global->increment('abc', 2));

var_export($global->cas('abc', 12, 18));

```

## Nota:
El componente GlobalData no puede compartir datos de tipo recurso, como conexiones mysql, conexiones de socket, etc.

Si se utiliza el cliente GlobalData/Client en un entorno Workerman, por favor instancia el objeto GlobalData/Client en los callbacks onXXX, por ejemplo en onWorkerStart.

No hagas operaciones de variable compartida de esta manera.
```php
$global->somekey = array();
$global->somekey[]='xxx';

$global->someObject = new someClass();
$global->someObject->someVar = 'xxx';
```
Puedes hacerlo de esta manera
```php
$somekey = array();
$somekey[] = 'xxx';
$global->somekey = $somekey;

$someObject = new someClass();
$someObject->someVar = 'xxx';
$global->someObject = $someObject;
```
