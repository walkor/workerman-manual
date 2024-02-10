# Protocolo ws

Atualmente, a versão do protocolo **ws do Workerman é 13**.

O Workerman pode ser usado como cliente, utilizando o protocolo ws para iniciar uma conexão de WebSocket e se conectar a um servidor remoto de WebSocket, realizando comunicação bidirecional.

> **Observação**
> O protocolo ws só pode ser utilizado como cliente através da AsyncTcpConnection e não pode ser utilizado como protocolo de escuta do servidor de WebSocket. Em outras palavras, a seguinte forma de escrita está incorreta.

```php
$worker = new Worker('ws://0.0.0.0:8080');
```

Se desejar usar o Workerman como servidor de WebSocket, por favor, utilize o [protocolo websocket](about-websocket.md).

**Exemplo do protocolo ws como cliente de WebSocket:**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// Ao iniciar o processo
$worker->onWorkerStart = function()
{
    // Conectar ao servidor remoto de WebSocket usando o protocolo websocket
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // Enviar um ping websocket para o servidor a cada 55 segundos com um opcode 0x9 (opcional)
    $ws_connection->websocketPingInterval = 55;
    // Definir cabeçalhos HTTP (opcional)
    $ws_connection->headers = [
        'Cookie' => 'PHPSID=82u98fjhakfusuanfnahfi; token=2hf9a929jhfihaf9i',
        'OtherKey' => 'values'
    ];
    // Definir tipo de dado (opcional)
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB para texto e BINARY_TYPE_ARRAYBUFFER para binário
    // Após o handshake do TCP (opcional)
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // Após o handshake do WebSocket (opcional)
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // Quando uma mensagem é recebida do servidor de WebSocket remoto
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // Em caso de erro na conexão, geralmente ocorre um erro na conexão com o servidor remoto de WebSocket (opcional)
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // Ao desconectar da conexão com o servidor remoto de WebSocket (opcional, recomenda-se adicionar uma tentativa de reconexão)
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // Se a conexão for encerrada, reconectar após 1 segundo
        $connection->reConnect(1);
    };
    // Após configurar todos esses callbacks, realizar a conexão
    $ws_connection->connect();
};
Worker::runAll();
```

Para mais informações, consulte [Como cliente ws/wss](../faq/as-wss-client.md).
