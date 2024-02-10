# Using Workerman as a ws/wss client

Sometimes it is necessary to use Workerman as a client to connect to a server using the ws/wss protocol and interact with it. Below are examples.

## Using Workerman as a ws client

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // After successful websocket handshake
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // When receiving a message
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Using Workerman as a wss (ws+ssl) client

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // SSL requires accessing port 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Set the connection to use SSL to become wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Using Workerman as a wss (ws+ssl) client with local SSL certificate

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Set the local IP and port to access the remote host with the SSL certificate
    $context_option = array(
        'ssl' => array(
            'local_cert'        => '/your/path/to/pemfile',
            'passphrase'        => 'your_pem_passphrase',
            'allow_self_signed' => true,
            'verify_peer'       => false
        )
    );

    // SSL requires accessing port 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Set the connection to use SSL to become wss
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Other settings

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();

$worker->onWorkerStart = function()
{
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    $ws_connection->websocketPingInterval = 55;
    $ws_connection->headers = ['token' => 'value'];
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB;
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        $connection->reConnect(1);
    };
    $ws_connection->connect();
};
Worker::runAll();
```
