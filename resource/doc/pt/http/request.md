# Documentação do Webman

O Webman fortaleceu o suporte para serviços HTTP a partir da versão 4.x do Workerman. Introduziu as classes de solicitação, resposta, sessão e [SSE](SSE.md). Se você deseja usar os serviços HTTP do Workerman, é altamente recomendável usar a versão 4.x do Workerman ou superior.

**Observe que todos os exemplos abaixo são utilizados na versão 4.x do Workerman e não são compatíveis com a versão 3.x.**

## Obter o objeto de solicitação
O objeto de solicitação sempre é obtido na função de retorno `onMessage`, o framework irá automaticamente passar o objeto Request como o segundo parâmetro para a função de retorno.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // $request é o objeto de solicitação, aqui não estamos realizando nenhuma operação no objeto de solicitação e simplesmente retornando "hello" para o navegador
    $connection->send("hello");
};

// Executar o worker
Worker::runAll();
```

Quando o navegador acessar `http://127.0.0.1:8080`, será retornado "hello".

## Obter parâmetros da solicitação GET
**Obter o array GET completo**
```php
$get = $request->get();
```
Se a solicitação não contiver parâmetros GET, ele retornará um array vazio.

**Obter um valor do array GET**
```php
$name = $request->get('name');
```
Se o array GET não contém o valor, ele retornará null.

Também é possível fornecer um valor padrão como segundo argumento para o método `get`. Se o valor correspondente não for encontrado no array GET, ele retornará o valor padrão. Por exemplo:
```php
$name = $request->get('name', 'tom');
```

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->get('name'));
};

// Executar o worker
Worker::runAll();
```

Quando o navegador acessar `http://127.0.0.1:8080?name=jerry&age=12`, será retornado "jerry".

## Obter parâmetros da solicitação POST
**Obter o array POST completo**
```php
$post = $request->post();
```
Se a solicitação não contiver parâmetros POST, ele retornará um array vazio.

**Obter um valor do array POST**
```php
$name = $request->post('name');
```
Se o array POST não contém o valor, ele retornará null.

Assim como o método `get`, também é possível fornecer um valor padrão como segundo argumento para o método `post`. Se o valor correspondente não for encontrado no array POST, ele retornará o valor padrão. Por exemplo:
```php
$name = $request->post('name', 'tom');
```

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = $request->post();
    $connection->send(var_export($post, true));
};

// Executar o worker
Worker::runAll();
```


## Obter o corpo da solicitação POST original
```php
$post = $request->rawBody();
```
Essa função é semelhante à operação `file_get_contents("php://input")` no `php-fpm` e é usada para obter o corpo da solicitação original HTTP. Isso é útil ao obter dados de solicitação POST em formato não `application/x-www-form-urlencoded`.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $post = json_decode($request->rawBody());
    $connection->send('hello');
};

// Executar o worker
Worker::runAll();
```

## Obter cabeçalhos (headers)
**Obter o array completo de cabeçalhos**
```php
$headers = $request->header();
```
Se a solicitação não contiver cabeçalhos, ele retornará um array vazio. Observe que todas as chaves são em letra minúscula.

**Obter um valor do array de cabeçalhos**
```php
$host = $request->header('host');
```
Se o array de cabeçalhos não contém o valor, ele retornará null. Observe que todas as chaves são em letra minúscula.

Da mesma forma que o método `get`, também é possível fornecer um valor padrão como segundo argumento para o método `header`. Se o valor correspondente não for encontrado no array de cabeçalhos, ele retornará o valor padrão. Por exemplo:
```php
$host = $request->header('host', 'localhost');
```
**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    if ($request->header('connection') === 'keep-alive') {
        $connection->send('hello');
    } else {
        $connection->close('hello');
    }    
};

// Executar o worker
Worker::runAll();
```

## Obter cookies
**Obter o array completo de cookies**
```php
$cookies = $request->cookie();
```
Se a solicitação não contiver cookies, ele retornará um array vazio.

**Obter um valor do array de cookies**
```php
$name = $request->cookie('name');
```
Se o array de cookies não contiver o valor, ele retornará null.

Da mesma forma que o método `get`, também é possível fornecer um valor padrão como segundo argumento para o método `cookie`. Se o valor correspondente não for encontrado no array de cookies, ele retornará o valor padrão. Por exemplo:
```php
$name = $request->cookie('name', 'tom');
```
**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $cookie = $request->cookie();
    $connection->send(var_export($cookie, true));
};

// Executar o worker
Worker::runAll();
```

## Obter arquivos enviados
**Obter o array completo de arquivos enviados**
```php
$files = $request->file();
```
O formato do array retornado é semelhante ao seguinte:
```php
array (
    'avatar' => array (
            'name' => '123.jpg',
            'tmp_name' => '/tmp/workerman.upload.9hjR4w',
            'size' => 1196127,
            'error' => 0,
            'type' => 'application/octet-stream',
      ),
     'anotherfile' =>  array (
            'name' => '456.txt',
            'tmp_name' => '/tmp/workerman.upload.9sirSws',
            'size' => 490,
            'error' => 0,
            'type' => 'text/plain',
      )
)
```
Onde:

 - `name` é o nome do arquivo
 - `tmp_name` é a localização do arquivo temporário no disco
 - `size` é o tamanho do arquivo
 - `error` é o [código de erro](https://www.php.net/manual/pt_BR/features.file-upload.errors.php)
 - `type` é o tipo MIME do arquivo.

**Observações:**

 - O tamanho dos arquivos enviados está sujeito à limitação do [defaultMaxPackageSize](../tcp-connection/default-max-package-size.md), padrão é 10 MB e pode ser modificado.

 - Os arquivos serão automaticamente excluídos após o término da solicitação.

 - Se a solicitação não contiver arquivos enviados, ele retornará um array vazio.

### Obter um arquivo de envio específico
```php
$avatar_file = $request->file('avatar');
```
Ele retornará algo semelhante a:
```php
array (
        'name' => '123.jpg',
        'tmp_name' => '/tmp/workerman.upload.9hjR4w',
        'size' => 1196127,
        'error' => 0,
        'type' => 'application/octet-stream',
  )
```
Se o arquivo de envio não existir, ele retornará null.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $file = $request->file('avatar');
    if ($file && $file['error'] === UPLOAD_ERR_OK) {
        rename($file['tmp_name'], '/home/www/web/public/123.jpg');
        $connection->send('ok');
        return;
    }
    $connection->send('upload fail');
};

// Executar o worker
Worker::runAll();
```

## Obter o host
Obtém as informações do host da solicitação.
```php
$host = $request->host();
```
Se o endereço da solicitação não for o padrão 80 ou 443, as informações do host podem conter a porta, por exemplo, `example.com:8080`. Se a porta não for necessária, o primeiro argumento pode receber `true`.

```php
$host = $request->host(true);
```
**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->host());
};

// Executar o worker
Worker::runAll();
```
Quando o navegador acessar `http://127.0.0.1:8080?name=tom`, retornará `127.0.0.1:8080`.
## Obter método de requisição
```php
$method = $request->method();
```
O valor de retorno pode ser `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`, ou `HEAD`.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->method());
};

// Executar o worker
Worker::runAll();
```

## Obter uri da requisição
```php
$uri = $request->uri();
```
Retorna a uri da requisição, incluindo a parte do caminho (path) e da query string.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->uri());
};

// Executar o worker
Worker::runAll();
```
Quando o navegador acessa `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, irá retornar `/user/get.php?uid=10&type=2`.

## Obter caminho da requisição
```php
$path = $request->path();
```
Retorna a parte do caminho (path) da requisição.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->path());
};

// Executar o worker
Worker::runAll();
```
Quando o navegador acessa `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, irá retornar `/user/get.php`.

## Obter query string da requisição
```php
$query_string = $request->queryString();
```
Retorna a parte da query string da requisição.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->queryString());
};

// Executar o worker
Worker::runAll();
```
Quando o navegador acessa `http://127.0.0.1:8080/user/get.php?uid=10&type=2`, irá retornar `uid=10&type=2`.

## Obter versão do HTTP da requisição
```php
$version = $request->protocolVersion();
```
Retorna a string `1.1` ou `1.0`.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->protocolVersion());
};

// Executar o worker
Worker::runAll();
```

## Obter sessionId da requisição
```php
$sid = $request->sessionId();
```
Retorna uma string composta por letras e números.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->sessionId());
};

// Executar o worker
Worker::runAll();
```

