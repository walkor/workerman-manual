# Método __construct
```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```
Cria um objeto de conexão assíncrona.

AsyncTcpConnection permite que o Workerman atue como cliente e inicie uma conexão assíncrona com um servidor remoto, e envie e processe dados de forma assíncrona através da interface send e do callback onMessage.

## Parâmetros
Parâmetro: `remote_address`

Endereço da conexão, por exemplo
``` tcp://www.baidu.com:80 ```
``` ssl://www.baidu.com:443 ```
``` ws://echo.websocket.org:80 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

Parâmetro: `$context_option`

`Este parâmetro requer (workerman >= 3.3.5)`

É utilizado para configurar o contexto do socket, por exemplo, usando `bindto` para definir a qual (placa de rede) IP e porta acessar a rede externa, configurar certificados SSL, etc.

Consulte [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [Opções de contexto de soquete](https://php.net/manual/zh/context.socket.php), [Opções de contexto SSL](https://php.net/manual/zh/context.ssl.php)

## Observação

Atualmente, AsyncTcpConnection suporta os protocolos [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md).

Também suporta protocolos personalizados, consulte [Como criar protocolos personalizados](../protocols/how-protocols.md)

O protocolo [ssl](https://baike.baidu.com/view/525499.htm) requer Workerman >= 3.3.4 e a instalação da extensão [openssl](https://php.net/manual/zh/book.openssl.php).

Atualmente, AsyncTcpConnection não suporta o protocolo [http](https://baike.baidu.com/view/9472.htm).

É possível usar `new AsyncTcpConnection('ws://...')` para iniciar uma conexão de WebSocket remota, semelhante a um navegador, no Workerman, consulte o [exemplo](../appendices/about-ws.md). No entanto, não é possível usar `new AsyncTcpConnection('websocket://...')` para iniciar uma conexão WebSocket no Workerman.

## Exemplos

### Exemplo 1: Acessar um serviço http externo de forma assíncrona
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Ao inicializar o processo, estabelece uma conexão assíncrona com www.baidu.com e envia dados para obter informações
$task->onWorkerStart = function($task)
{
    // Não suporta a especificação direta de http, mas é possível simular o envio do protocolo http usando o protocolo tcp
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // Quando a conexão é estabelecida com sucesso, envia os dados da requisição http
    $connection_to_baidu->onConnect = function (AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexão estabelecida com sucesso\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function (AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function (AsyncTcpConnection $connection_to_baidu)
    {
        echo "Conexão encerrada\n";
    };
    $connection_to_baidu->onError = function (AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "Código de erro: $code, mensagem: $msg\n";
    };
    $connection_to_baidu->connect();
};

// Executa o worker
Worker::runAll();
```

### Exemplo 2: Acessar um serviço de WebSocket externo de forma assíncrona e definir a qual ip local e porta acessar
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Define o IP local e a porta para acessar o host remoto (cada conexão de socket ocupará uma porta local)
    $context_option = array(
        'socket' => array(
            // O IP deve ser um IP da placa de rede local, e deve ser capaz de acessar o host remoto, caso contrário será inválido
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

### Exemplo 3: Acessar de forma assíncrona uma porta wss externa e configurar um certificado ssl local
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // Define o IP local e a porta para acessar o host remoto, e o certificado SSL local
    $context_option = array(
        'socket' => array(
            // O IP deve ser um IP da placa de rede local, e deve ser capaz de acessar o host remoto, caso contrário será inválido
            'bindto' => '114.215.84.87:2333',
        ),
        // Opções ssl, consulte https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // Caminho do certificado local. Deve ser no formato PEM e conter o certificado local e a chave privada.
            'local_cert'        => '/seu/caminho/para/arquivo.pem',
            // Senha do arquivo local_cert.
            'passphrase'        => 'sua_senha_pem',
            // Permitir ou não certificados auto-assinados.
            'allow_self_signed' => true,
            // Se a verificação do certificado SSL é necessária.
            'verify_peer'       => false
        )
    );

    // Inicia a conexão assíncrona
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // Define o método de acesso como SSL
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
