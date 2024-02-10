# Persistencia de objetos y recursos
En el desarrollo web tradicional, los objetos, datos, recursos, etc. creados por PHP se liberarán por completo una vez completada la solicitud, lo que hace que sea difícil lograr la persistencia. Sin embargo, con Webman, esto se puede lograr con facilidad.

En Webman, si se desea mantener permanentemente ciertos recursos en la memoria, se pueden colocar en variables globales o miembros estáticos de una clase.

Por ejemplo, en el siguiente código:

Se utiliza una variable global ```$connection_count``` para almacenar el número de conexiones de clientes en el proceso actual.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Variable global para mantener el número de conexiones de clientes en el proceso actual
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // Aumentar el número de conexiones cuando se establece una nueva conexión de cliente
    global $connection_count;
    ++$connection_count;
    echo "now connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // Reducir el número de conexiones cuando se cierra un cliente
    global $connection_count;
    $connection_count--;
    echo "now connection_count=$connection_count\n";
};
```

## Consulta el ámbito de las variables en PHP en:
https://php.net/manual/zh/language.variables.scope.php
