# transporte
## Descripción:
```php
string Worker::$transport
```

Establece el protocolo de transporte que se utiliza para la instancia actual del Worker. Actualmente, solo se admiten tres tipos (tcp, udp, ssl). Si no se establece, el valor predeterminado es tcp.

```Nota: ssl requiere Workerman versión >= 3.3.7```

## Ejemplo 1
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Usar el protocolo udp
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hola');
};
// Ejecutar el worker
Worker::runAll();
```

## Ejemplo 2
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// El certificado debería ser preferiblemente uno emitido por una entidad de confianza
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // también puede ser un archivo crt
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Aquí se establece el protocolo websocket
$worker = new Worker('websocket://0.0.0.0:4331', $context);
// Establecer el transporte para activar ssl, websocket+ssl es wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
