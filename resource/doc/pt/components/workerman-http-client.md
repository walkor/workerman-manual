# workerman/http-client
## Descrição
 [workerman/http-client](https://github.com/walkor/http-client) é um componente de cliente HTTP assíncrono. Todas as solicitações e respostas são assíncronas e não bloqueantes, com pool de conexões embutido, e as mensagens de solicitação e resposta estão de acordo com a especificação PSR7.

## Instalação:
```php
composer require workerman/http-client
```

## Exemplo:

**Uso de solicitações GET e POST**

```php
use Workerman\Worker;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();

    $http->get('https://example.com/', function ($response) {
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function ($exception) {
        echo $exception;
    });

    $http->post('https://example.com/', ['key1' => 'value1', 'key2' => 'value2'], function ($response) {
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function ($exception) {
        echo $exception;
    });

    $http->request('https://example.com/', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive'],
        'data' => ['key1' => 'value1', 'key2' => 'value2'],
        'success' => function ($response) {
            echo $response->getBody();
        },
        'error' => function ($exception) {
            echo $exception;
        }
    ]);
};
Worker::runAll();
```

**Enviar arquivo**

```php
<?php
use Workerman\Worker;

require_once 'vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();
    // Envio de arquivo
    $multipart = new \Workerman\Psr7\MultipartStream([
        [
            'name' => 'file',
            'contents' => fopen(__FILE__, 'r')
        ],
        [
            'name' => 'json',
            'contents' => json_encode(['a'=>1, 'b'=>2])
        ]
    ]);
    $boundary = $multipart->getBoundary();
    $http->request('http://127.0.0.1:8787', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive', 'Content-Type' => "multipart/form-data; boundary=$boundary"],
        'data' => $multipart,
        'success' => function ($response) {
            echo $response->getBody();
        },
        'error' => function ($exception) {
            echo $exception;
        }
    ]);
};

Worker::runAll();
```

**Retorno do fluxo de progresso**

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Http\Client;
use Workerman\Protocols\Http\Chunk;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Worker;

$worker = new Worker('http://0.0.0.0:1234');
$worker->onMessage = function (TcpConnection $connection, Request $request) {
    $http = new Client();
    $http->request('https://api.bla.cn/v1/chat/completions', [
        'method' => 'POST',
        'data' => json_encode([
            'model' => 'gpt-3.5-turbo',
            'temperature' => 1,
            'stream' => true,
            'messages' => [['role' => 'user', 'content' => 'hello']],
        ]),
        'headers' => [
            'Content-Type' => 'application/json',
            'Authorization' => 'Bearer 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',
        ],
        'progress' => function($buffer) use ($connection) {
            $connection->send(new Chunk($buffer));
        },
        'success' => function($response) use ($connection) {
            $connection->send(new Chunk(''));
        },
    ]);
    $connection->send(new Response(200, [
        //"Content-Type" => "application/octet-stream",
        "Transfer-Encoding" => "chunked",
    ], ' '));
};
Worker::runAll();
```

## Opções
```php
<?php
require __DIR__ . '/vendor/autoload.php';
use Workerman\Worker;
$worker = new Worker();
$worker->onWorkerStart = function(){
    $options = [
        'max_conn_per_addr' => 128, // Número máximo de conexões simultâneas por domínio
        'keepalive_timeout' => 15,  // Fechar a conexão após determinado tempo sem comunicação
        'connect_timeout'   => 30,  // Tempo limite de conexão
        'timeout'           => 30,  // Tempo limite de espera pela resposta após o envio da solicitação
    ];
    $http = new Workerman\Http\Client($options);

    $http->get('http://example.com/', function($response){
        var_dump($response->getStatusCode());
        echo $response->getBody();
    }, function($exception){
        echo $exception;
    });
};
Worker::runAll();
```

## Uso de Corotina

> **Nota**
> O uso de corotina requer workerman>=5.0, workerman/http-client>=2.0.0 e a instalação do composer require revolt/event-loop ^1.0.0

```php
use Workerman\Worker;

require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function () {
    $http = new Workerman\Http\Client();

    $response = $http->get('https://example.com/');
    var_dump($response->getStatusCode());
    echo $response->getBody();

    $response = $http->post('https://example.com/', ['key1' => 'value1', 'key2' => 'value2']);
    var_dump($response->getStatusCode());
    echo $response->getBody();
    

    $response = $http->request('https://example.com/', [
        'method' => 'POST',
        'version' => '1.1',
        'headers' => ['Connection' => 'keep-alive'],
        'data' => ['key1' => 'value1', 'key2' => 'value2'],
    ]);
    echo $response->getBody();
};
Worker::runAll();
```

Quando nenhuma função de retorno é definida, o cliente retornará o resultado da solicitação de forma síncrona de maneira assíncrona, sem bloquear o processo atual, ou seja, pode lidar com solicitações concorrentes.

## Observações:

1. O projeto precisa carregar inicialmente `require __DIR__ . '/vendor/autoload.php';`
2. Todo o código assíncrono só pode ser executado após a inicialização do ambiente de execução do workerman
3. Suporta todos os projetos baseados no desenvolvimento do workerman, incluindo GatewayWorker, PHPSocket.io, entre outros.
