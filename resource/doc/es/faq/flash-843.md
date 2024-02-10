# Habilitar el puerto 843 para Flash

Cuando Flash intenta establecer una conexión de socket con un servidor remoto, primero solicita un archivo de política de seguridad en el puerto 843 del servidor correspondiente. De lo contrario, Flash no puede establecer la conexión con el servidor. En Workerman, se puede abrir un puerto 843 de la siguiente manera para enviar el archivo de política de seguridad.

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$flash_policy = new Worker('tcp://0.0.0.0:843');
$flash_policy->onMessage = function(TcpConnection $connection, $message)
{
    $connection->send('<?xml version="1.0"?><cross-domain-policy><site-control permitted-cross-domain-policies="all"/><allow-access-from domain="*" to-ports="*"/></cross-domain-policy>'."\0");
};

if(!defined('GLOBAL_START'))
{
    Worker::runAll();
}
```

El contenido de la política de seguridad XML puede ser personalizado según sus necesidades.

