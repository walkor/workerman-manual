# Transport
## Description
```php
string Worker::$transport
```
Sets the transport layer protocol used by the current Worker instance. Currently, only 3 kinds are supported (tcp, udp, ssl). If not set, the default is tcp.

``` Note: ssl requires Workerman version >= 3.3.7 ```

## Example 1
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:8484');
// Use the udp protocol
$worker->transport = 'udp';
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('Hello');
};
// Run the worker
Worker::runAll();
```

## Example 2
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// The certificate should ideally be obtained from certificate authorities
$context = array(
    'ssl' => array(
        'local_cert' => '/etc/nginx/conf.d/ssl/server.pem', // It can also be a .crt file
        'local_pk'   => '/etc/nginx/conf.d/ssl/server.key',
    )
);
// Here we set the websocket protocol
$worker = new Worker('websocket://0.0.0.0:4431', $context);
// Set the transport layer to enable ssl, making it websocket+ssl (wss)
$worker->transport = 'ssl';
$worker->onMessage = function(TcpConnection $con, $msg) {
    $con->send('ok');
};

Worker::runAll();
```
