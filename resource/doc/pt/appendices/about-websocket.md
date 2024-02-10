# Protocolo WebSocket

Atualmente, a versão do protocolo WebSocket no Workerman é **13**.

O protocolo WebSocket é um novo protocolo da HTML5. Ele implementa a comunicação bidirecional entre o navegador e o servidor.

## Relação entre WebSocket e TCP

Assim como o HTTP, o WebSocket é um protocolo de aplicação que é baseado na transmissão TCP. O WebSocket em si não tem muita relação com Sockets e muito menos pode ser igualado a eles.

## Aperto de mão do Protocolo WebSocket

O protocolo WebSocket possui um processo de aperto de mão, durante o qual o navegador e o servidor comunicam-se usando o protocolo HTTP. No Workerman, é possível intervir nesse processo de aperto de mão da seguinte maneira.

**Quando o Workerman <= 4.1**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Worker;

$ws = new Worker('websocket://0.0.0.0:8181');
$ws->onConnect = function($connection)
{
    $connection->onWebSocketConnect = function($connection , $httpBuffer)
    {
        // Você pode verificar se a conexão é válida aqui e fechá-la se não for válida
        // $_SERVER['HTTP_ORIGIN'] indica de qual site a conexão WebSocket foi iniciada
        if($_SERVER['HTTP_ORIGIN'] != 'https://www.workerman.net')
        {
            $connection->close();
        }
        // Dentro do onWebSocketConnect $_GET $_SERVER estão disponíveis
        // var_dump($_GET, $_SERVER);
    };
};
Worker::runAll();
```

**Quando o Workerman >= 5.0**
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Worker;

$worker = new Worker('websocket://0.0.0.0:12345');
$worker->onWebSocketConnect = function (TcpConnection $connection, Request $request) {
    if ($request->header('origin') != 'https://www.workerman.net') {
        $connection->close();
    }
    var_dump($request->get());
    var_dump($request->header());
};
Worker::runAll();
```

## Transmissão de dados binários pelo Protocolo WebSocket

O protocolo WebSocket por padrão só pode transmitir texto UTF-8. Se deseja transmitir dados binários, leia a seguinte seção.

No protocolo WebSocket, um marcador é utilizado no cabeçalho para indicar se os dados transmitidos são binários ou texto UTF-8. O navegador irá verificar o marcador e o tipo de conteúdo transmitido. Em caso de não conformidade, a conexão será encerrada.

Portanto, ao enviar dados do servidor, é necessário definir esse marcador com base no tipo de dados sendo transmitidos no Workerman. Se forem dados de texto UTF-8 padrão, é necessário definir (geralmente este valor padrão é utilizado, e não precisa ser configurado manualmente)
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_BLOB;
```

Se forem dados binários, é necessário configurar
```php
use Workerman\Protocols\Websocket;
$connection->websocketType = Websocket::BINARY_TYPE_ARRAYBUFFER;
```

**Observação**: Se $connection->websocketType não for configurado, ele será, por padrão, BINARY_TYPE_BLOB (ou seja, tipo de texto UTF-8). Geralmente, aplicativos transmitem texto UTF-8, como dados JSON, então não é necessário configurar manualmente $connection->websocketType. Somente ao transmitir dados binários (por exemplo, dados de imagem, dados de protocolo buffer, etc.) é necessário configurar essa propriedade como BINARY_TYPE_ARRAYBUFFER.

## Usando o Workerman como cliente WebSocket
É possível utilizar a classe [AsyncTcpConnection](../async-tcp-connection.md) em conjunto com o protocolo [ws](about-ws.md) para fazer com que o Workerman atue como cliente WebSocket conectando-se a um servidor remoto de WebSocket, permitindo a comunicação em tempo real bidirecional.
