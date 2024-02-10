# __construct Method
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Creates an asynchronous connection object.

AsyncTcpConnection allows Workerman to initiate asynchronous connections to remote servers as a client, and to asynchronously send and process data on the connection using the `send` interface and `onMessage` callback.

## Parameters
Parameter: `remote_address`

The address of the connection, for example:
- `tcp://www.baidu.com:80`
- `ssl://www.baidu.com:443`
- `ws://echo.websocket.org:80`
- `frame://192.168.1.1:8080`
- `text://192.168.1.1:8080`

Parameter: `$context_option`

Requires this parameter (workerman >= 3.3.5).

Used to set the socket context, for example, using `bindto` to specify which (network card) IP and port to access the external network, setting SSL certificates, etc.

See [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Socket Context Options](https://php.net/manual/zh/context.socket.php), and [SSL Context Options](https://php.net/manual/zh/context.ssl.php).

## Note
Currently, AsyncTcpConnection supports protocols such as [tcp](https://en.wikipedia.org/wiki/Transmission_Control_Protocol), [ssl](https://en.wikipedia.org/wiki/Transport_Layer_Security), [ws](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), [frame](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), and [text](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API).

Custom protocols are also supported, see [How to Define Protocols](../protocols/how-protocols.md).

The use of [ssl](https://en.wikipedia.org/wiki/Transport_Layer_Security) requires Workerman >= 3.3.4 and the installation of the [openssl extension](https://php.net/manual/zh/book.openssl.php).

AsyncTcpConnection currently does not support the [http](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) protocol.

You can use `new AsyncTcpConnection('ws://...')` to initiate a WebSocket connection to a remote WebSocket server in Workerman, similar to a browser. Refer to the [example](../appendices/about-ws.md). However, it is not possible to initiate a WebSocket connection in Workerman using `new AsyncTcpConnection('websocket://...')`.

## Examples

### Example 1: Asynchronously access external HTTP service
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();

$task->onWorkerStart = function($task) {
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu) {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };

    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer) {
        echo $http_buffer;
    };

    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu) {
        echo "connection closed\n";
    };

    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg) {
        echo "Error code:$code msg:$msg\n";
    };

    $connection_to_baidu->connect();
};

Worker::runAll();
```

### Example 2: Asynchronously access external WebSocket service and set the local IP and port for access
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    $context_option = array(
        'socket' => array(
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### Example 3: Asynchronously access external wss port and set the local SSL certificate
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    $context_option = array(
        'socket' => array(
            'bindto' => '114.215.84.87:2333',
        ),
        'ssl' => array(
            'local_cert'        => '/your/path/to/pemfile',
            'passphrase'        => 'your_pem_passphrase',
            'allow_self_signed' => true,
            'verify_peer'       => false
        )
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

