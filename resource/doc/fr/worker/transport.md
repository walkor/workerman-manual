# transport
## Description:
```php
string Worker::$transport
```

Définit le protocole de transport utilisé par l'instance Worker actuelle. Actuellement, seules 3 options sont prises en charge (tcp, udp, ssl). Si aucune option n'est définie, la valeur par défaut est tcp.

``` Note: SSL nécessite Workerman version >= 3.3.7 ```

## Example 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Utiliser le protocole udp
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// Exécuter le worker
Worker::runAll();
```

## Example 2

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Il est préférable de disposer d'un certificat valide
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // Peut également être un fichier crt
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Ici, le protocole websocket est configuré
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// Définir le transport pour activer ssl, websocket+ssl est donc wss
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
