# Explicação

A partir da versão 4.x, o Workerman reforçou o suporte aos serviços HTTP. Introduziu classes de requisição, classes de resposta, classes de sessão e [SSE](SSE.md). Se você deseja utilizar os serviços HTTP do Workerman, é altamente recomendável usar a versão 4.x ou superior.

**Observe que todos os exemplos pertencem à versão 4.x do Workerman e não são compatíveis com a versão 3.x.**

# Obter objeto de sessão
```php
$session = $request->session();
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
    $session = $request->session();
    $session->set('name', 'tome');
    $connection->send($session->get('name'));
};

// Executar worker
Worker::runAll();
```
**Observações**
- A sessão deve ser manipulada antes de chamar `$connection->send()`.
- A sessão é automaticamente salva quando o objeto é destruído, portanto, não salve o objeto retornado por `$request->session()` em um array global ou membro da classe para evitar que a sessão não seja salva.
- Por padrão, a sessão é armazenada em arquivos no disco. Para obter melhor desempenho, é recomendável utilizar o Redis.

## Obter todos os dados da sessão
```php
$session = $request->session();
$all = $session->all();
```
Retorna um array. Se não houver dados de sessão, um array vazio é retornado.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send(var_export($session->all(), true));
};

// Executar worker
Worker::runAll();
```

## Obter um valor específico da sessão
```php
$session = $request->session();
$name = $session->get('name');
```
Se os dados não existirem, retorna null.

Também é possível passar um valor padrão como segundo argumento para o método `get`. Se o valor correspondente não for encontrado no array da sessão, o valor padrão é retornado. Por exemplo:
```php
$session = $request->session();
$name = $session->get('name', 'tom');
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
    $session = $request->session();
    $connection->send($session->get('name', 'tom'));
};

// Executar worker
Worker::runAll();
```

## Armazenar sessão
Para armazenar um valor, use o método `set`.
```php
$session = $request->session();
$session->set('name', 'tom');
```
O método `set` não retorna nada e a sessão é automaticamente salva quando o objeto da sessão é destruído.

Para armazenar múltiplos valores, use o método `put`.
```php
$session = $request->session();
$session->put(['name' => 'tom', 'age' => 12]);
```
Da mesma forma, o método `put` não retorna nada.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $session = $request->session();
    $session->set('name', 'tom');
    $connection->send($session->get('name'));
};

// Executar worker
Worker::runAll();
```

## Excluir dados da sessão
Para excluir um ou vários dados da sessão, use o método `forget`.
```php
$session = $request->session();
// Delete one
$session->forget('name');
// Delete multiple
$session->forget(['name', 'age']);
```
Além disso, o sistema fornece o método `delete`, que difere do `forget` ao excluir apenas um item.
```php
$session = $request->session();
// Equivalente a $session->forget('name');
$session->delete('name');
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
    $request->session()->forget('name');
    $connection->send('ok');
};

// Executar worker
Worker::runAll();
```

## Obter e excluir um valor da sessão
```php
$session = $request->session();
$name = $session->pull('name');
```
O efeito é o mesmo que o seguinte código:
```php
$session = $request->session();
$value = $session->get($name);
$session->delete($name);
```
Se a sessão correspondente não existir, retorna null.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $connection->send($request->session()->pull('name'));
};

// Executar worker
Worker::runAll();
```

## Excluir todos os dados da sessão
```php
$request->session()->flush();
```
Não retorna nada e a sessão é automaticamente removida do armazenamento quando o objeto de sessão é destruído.

**Exemplo**
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    $request->session()->flush();
    $connection->send('ok');
};

// Executar worker
Worker::runAll();
```

## Verificar se os dados da sessão correspondente existem
```php
$session = $request->session();
$has = $session->has('name');
```
Quando a sessão correspondente não existe ou o valor correspondente é nulo, retorna falso; caso contrário, retorna verdadeiro.

```
$session = $request->session();
$has = $session->exists('name');
```
Este código também é utilizado para verificar se os dados da sessão existem. A diferença é que, quando o valor do item da sessão correspondente é nulo, ainda retorna verdadeiro.
