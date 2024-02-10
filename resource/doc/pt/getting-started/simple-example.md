# Exemplo de Desenvolvimento Simples

## Instalação

**Instalar o Workerman**
Execute o seguinte comando em um diretório vazio
`composer require workerman/workerman`

## Exemplo 1: Fornecer Serviço Web Externo usando o protocolo HTTP

**Crie o arquivo start.php**
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

// Crie um Worker para ouvir a porta 2345 e usar comunicação do protocolo HTTP
$http_worker = new Worker("http://0.0.0.0:2345");

// Inicialize 4 processos para fornecer serviços externos
$http_worker->count = 4;

// Quando receber dados enviados pelo navegador, responderá com "hello world" para o navegador
$http_worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Enviar "hello world" para o navegador
    $connection->send('hello world');
};

// Execute o worker
Worker::runAll();
```

**Execute no terminal (para os usuários do Windows, use o [cmd](https://baike.baidu.com/item/%E5%91%BD%E4%BB%A4%E6%8F%90%E7%A4%BA%E7%AC%A6?fromtitle=CMD&fromid=1193011&type=syn))**
```shell
php start.php start
```

**Teste**

Supondo o endereço IP do servidor seja 127.0.0.1

Acesse a URL http://127.0.0.1:2345 em um navegador

**Nota:**

1. Se não for possível acessar, consulte a seção [Razões para Falha na Conexão do Cliente](../faq/client-connect-fail.md) para solucionar o problema.

2. O servidor é do protocolo HTTP, podendo apenas se comunicar utilizando o protocolo HTTP. Não é possível se comunicar diretamente com outros protocolos, como o WebSocket.

## Exemplo 2: Fornecer Serviço Externo utilizando o protocolo WebSocket

**Crie o arquivo ws_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Observação: diferente do exemplo anterior, aqui usaremos o protocolo websocket
$ws_worker = new Worker("websocket://0.0.0.0:2000");

// Inicialize 4 processos para fornecer serviços externos
$ws_worker->count = 4;

// Quando receber dados enviados pelo cliente, retornará "hello $data" para o cliente
$ws_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Enviar "hello $data" para o cliente
    $connection->send('hello ' . $data);
};

// Execute o worker
Worker::runAll();
```

**Execute no terminal**
```shell
php ws_test.php start
```

**Teste**

Abra o navegador Chrome e pressione F12 para abrir o console de inspeção. Na aba Console, digite (ou insira o código abaixo em uma página HTML e execute-o em JavaScript)

```javascript
// Supondo o endereço IP do servidor seja 127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2000");
ws.onopen = function() {
    alert("Conexão bem-sucedida");
    ws.send('tom');
    alert("Envie uma string para o servidor: tom");
};
ws.onmessage = function(e) {
    alert("Mensagem recebida do servidor: " + e.data);
};
```

**Nota:**

1. Se não for possível acessar, consulte a seção [Falha na Conexão do Cliente](../faq/client-connect-fail.md) no Manual de Perguntas Frequentes para solucionar o problema.

2. O servidor é do protocolo WebSocket, podendo apenas se comunicar utilizando o protocolo WebSocket. Não é possível se comunicar diretamente com outros protocolos, como o HTTP.

## Exemplo 3: Transferência Direta de Dados via TCP

**Crie o arquivo tcp_test.php**

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Crie um Worker para ouvir a porta 2347 sem utilizar nenhum protocolo de camada de aplicativo
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// Inicialize 4 processos para fornecer serviços externos
$tcp_worker->count = 4;

// Quando o cliente enviar dados
$tcp_worker->onMessage = function(TcpConnection $connection, $data)
{
    // Enviar "hello $data" para o cliente
    $connection->send('hello ' . $data);
};

// Execute o worker
Worker::runAll();
```

**Execute no terminal**
```shell
php tcp_test.php start
```

**Teste: Execute no terminal**
(Aqui está a saída do terminal Linux, a aparência é um pouco diferente em ambientes Windows)

```shell
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
tom
hello tom
```

**Nota:**

1. Se não for possível acessar, consulte a seção [Falha na Conexão do Cliente](../faq/client-connect-fail.md) no Manual de Perguntas Frequentes para solucionar o problema.

2. O servidor é do protocolo TCP bruto, não é possível se comunicar diretamente com outros protocolos como WebSocket e HTTP.
