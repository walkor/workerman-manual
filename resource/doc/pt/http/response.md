# Introdução

A partir da versão 4.x, o workerman reforçou o suporte ao serviço HTTP. Introduziu classes de requisição, resposta, sessão e [SSE](SSE.md). Se você deseja usar o serviço HTTP do workerman, é altamente recomendável usar a versão 4.x ou posterior.

**Observe que os seguintes exemplos são para uso da versão 4.x do workerman e não são compatíveis com a versão 3.x.**

# Atenção

- A menos que esteja enviando uma resposta em chunks ou SSE, não é permitido enviar a resposta várias vezes em uma única solicitação, ou seja, não é permitido chamar `$connection->send()` várias vezes em uma única solicitação.
- Cada solicitação deve chamar `$connection->send()` pelo menos uma vez para enviar a resposta, caso contrário o cliente ficará aguardando indefinidamente.

## Resposta Rápida
Quando não for necessário alterar o código de status HTTP (padrão 200), ou personalizar cabeçalhos e cookies, você pode enviar uma string diretamente para o cliente para concluir a resposta.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Envie diretamente "this is body" para o cliente
    $connection->send("this is body");
};

// Executar o worker
Worker::runAll();
```

## Alterando o Código de Status
Quando for necessário personalizar o código de status, cabeçalhos e cookies, é necessário usar a classe de resposta `Workerman\Protocols\Http\Response`. Por exemplo, no seguinte exemplo, ao acessar o caminho `/404`, o código de status 404 será retornado com o conteúdo `<h1>Desculpe, o arquivo não existe</h1>`.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->path() === '/404') {
        $connection->send(new Response(404, [], '<h1>Desculpe, o arquivo não existe</h1>'));
    } else {
        $connection->send('this is body');
    }
};

// Executar o worker
Worker::runAll();
```
Depois que a classe `Response` é inicializada, é possível modificar o código de status usando o método abaixo.
```php
$response = new Response(200);
$response->withStatus(404);
$connection->send($response);
```

## Enviando Cabeçalhos
Da mesma forma, para enviar cabeçalhos é necessário usar a classe de resposta `Workerman\Protocols\Http\Response`.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [
        'Content-Type' => 'text/html',
        'X-Header-One' => 'Valor do Cabeçalho'
    ], 'this is body');
    $connection->send($response);
};

// Executar o worker
Worker::runAll();
```
Depois que a classe `Response` é inicializada, é possível adicionar ou modificar cabeçalhos usando o método abaixo.
```php
$response = new Response(200);
// Adicionar ou modificar um cabeçalho
$response->header('Content-Type', 'text/html');
// Adicionar ou modificar vários cabeçalhos
$response->withHeaders([
    'Content-Type' => 'application/ json',
    'X-Header-One' => 'Valor do Cabeçalho'
]);
$connection->send($response);
```

## Redirecionamento
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)

$worker = new Worker('http://0.0.0.0:8080');
$worker->onMessage = function($connection, $request)
{
    $location = '/test_location';
    $response = new Response(302, ['Location' => $location]);
    $connection->send($response);
};
Worker::runAll();
```

## Enviando Cookie
Da mesma forma, para enviar um cookie é necessário usar a classe de resposta `Workerman\Protocols\Http\Response`.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $response = new Response(200, [], 'this is body');
    $response->cookie('name', 'tom');
    $connection->send($response);
};

// Executar o worker
Worker::runAll();
```

## Enviando Arquivo
Da mesma forma, para enviar um arquivo é necessário usar a classe de resposta `Workerman\Protocols\Http\Response`.

Para enviar um arquivo, use o seguinte método
```php
$response = (new Response())->withFile($file);
$connection->send($response);
```
- O workerman suporta o envio de arquivos muito grandes.
- Para arquivos grandes (maiores que 2M), o workerman não carrega o arquivo inteiro na memória de uma vez, mas o lê e envia em partes no momento apropriado.
- O workerman otimiza a velocidade de leitura e envio do arquivo com base na velocidade de recebimento do cliente, garantindo o envio mais rápido do arquivo enquanto reduz o uso de memória ao mínimo.
- A transmissão de dados é não-bloqueante e não afeta o processamento de outras solicitações.
- Ao enviar um arquivo, o cabeçalho `Last-Modified` é automaticamente adicionado para que o servidor possa decidir se enviar uma resposta 304 na próxima solicitação, poupando a transmissão do arquivo e melhorando o desempenho.
- O arquivo enviado automaticamente utilizará o cabeçalho `Content-Type` apropriado para enviar ao navegador.
- Se o arquivo não existir, será automaticamente tratado como uma resposta 404.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = '/seu/caminho/do/arquivo';
    // Verifique se o cabeçalho if-modified-since para verificar se o arquivo foi modificado
    if (!empty($if_modified_since = $request->header('if-modified-since'))) {
        $modified_time = date('D, d M Y H:i:s',  filemtime($file)) . ' ' . \date_default_timezone_get();
        // Se o arquivo não foi modificado, retorne 304
        if ($modified_time === $if_modified_since) {
            $connection->send(new Response(304));
            return;
        }
    }
    // Se o arquivo foi modificado ou o cabeçalho if-modified-since está ausente, envie o arquivo
    $response = (new Response())->withFile($file);
    $connection->send($response);
};

// Executar o worker
Worker::runAll();
```

## Enviar dados de chunk HTTP
- É necessário enviar uma resposta inicial com cabeçalho `Transfer-Encoding: chunked` para o cliente.
- Para enviar dados de chunk subsequentes, use a classe `Workerman\Protocols\Http\Chunk`.
- No final, é necessário enviar um chunk vazio para encerrar a resposta.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\Response;
use Workerman\Protocols\Http\Chunk;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Envie inicialmente uma resposta com o cabeçalho Transfer-Encoding: chunked
    $connection->send(new Response(200, array('Transfer-Encoding' => 'chunked'), 'hello'));
    // Use a classe Workerman\Protocols\Http\Chunk para enviar dados de chunk subsequentes
    $connection->send(new Chunk('Dados do primeiro bloco'));
    $connection->send(new Chunk('Dados do segundo bloco'));
    $connection->send(new Chunk('Dados do terceiro bloco'));
   //  Finalmente, é necessário enviar um chunk vazio para encerrar a resposta
    $connection->send(new Chunk(''));
};

// Executar o worker
Worker::runAll();
```
