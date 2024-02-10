# Como cliente de ws/wss

Às vezes, é necessário que o workerman atue como cliente usando o protocolo ws/wss para se conectar a um servidor e interagir com ele. Abaixo está um exemplo.

## Workerman como cliente de ws

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // Após a conexão bem-sucedida do websocket
    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    // Quando uma mensagem é recebida
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## Workerman como cliente de wss (ws+ssl)

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // O SSL precisa acessar a porta 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // Configurar a conexão para usar SSL e se tornar wss
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


## Workerman como cliente de wss (ws+ssl) com certificado SSL local

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Configurar o IP local, porta e certificado SSL do host remoto
    $context_option = array(
        // Opções SSL, consulte http://php.net/manual/pt_BR/context.ssl.php
        'ssl' => array(
            // Caminho do certificado local. Deve estar no formato PEM e incluir o certificado local e a chave privada.
            'local_cert'        => '/your/path/to/pemfile',
            // Senha do arquivo local_cert.
            'passphrase'        => 'your_pem_passphrase',
            // Permitir certificados autoassinados.
            'allow_self_signed' => true,
            // Verificar o certificado SSL é necessário.
            'verify_peer'       => false
        )
    );

    // O SSL precisa acessar a porta 443
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Configurar a conexão para usar SSL e se tornar wss
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

## Outras configurações

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Quando o processo é iniciado
$worker->onWorkerStart = function()
{
    // Conectar-se a um servidor remoto usando o protocolo websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Enviar um ping websocket opcode 0x9 para o servidor a cada 55 segundos
    $ws_connection->websocketPingInterval = 55;
    // Cabeçalhos HTTP personalizados
    $ws_connection->headers = ['token' => 'value'];
    // Definir o tipo de dados. Por padrão, BINARY_TYPE_BLOB é usado para texto
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB para texto, BINARY_TYPE_ARRAYBUFFER para binário
    // Após a conclusão do handshake TCP
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // Após a conclusão do handshake websocket
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Quando o servidor remoto envia uma mensagem
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // Em caso de erro na conexão, geralmente ao falhar a conexão com o servidor websocket remoto
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Quando a conexão com o servidor websocket remoto é encerrada
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // Se a conexão for encerrada, reconectar após 1 segundo
        $connection->reConnect(1);
    };
    // Após configurar todas as chamadas de retorno acima, realizar a conexão
    $ws_connection->connect();
};
Worker::runAll();
```
