# Persistencia de objetos y recursos
En el desarrollo web tradicional, los objetos, datos y recursos creados por PHP se liberan por completo después de que se completa la solicitud, lo que hace que sea difícil lograr la persistencia. Sin embargo, con Workerman, es posible lograr esto de manera sencilla.

En Workerman, si se desea mantener permanentemente ciertos datos o recursos en memoria, se pueden colocar en una variable global o en miembros estáticos de una clase.

Por ejemplo, en el siguiente código:

Se utiliza la variable global ```$connection_count``` para guardar el número de conexiones de clientes en el proceso actual.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Variable global para guardar el número de conexiones de clientes en el proceso actual
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // Cuando se establece una nueva conexión de cliente, se incrementa el contador de conexiones
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // Cuando se cierra un cliente, se reduce el contador de conexiones
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};

```

## Consulta el ámbito de variables en PHP en:
https://php.net/manual/zh/language.variables.scope.php
