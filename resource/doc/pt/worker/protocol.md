# protocol
Requerido ```(workerman >= 3.2.7)```

## Descrição:
```php
string Worker::$protocol
```

Define a classe de protocolo para a instância atual do Worker.

Nota: A classe de processamento de protocolo pode ser especificada diretamente ao inicializar o Worker no parâmetro de escuta. Por exemplo:
```php
$worker = new Worker('http://0.0.0.0:8686');
```

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8686');
$worker->protocol = 'Workerman\\Protocols\\Http';

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Executar o worker
Worker::runAll();
```

O código acima é equivalente ao código abaixo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

/**
 * Primeiramente, o sistema verifica se o usuário possui uma classe de protocolo personalizada \Protocols\Http.
 * Se não houver, será utilizado a classe de protocolo interna do workerman Workerman\Protocols\Http.
 */
$worker = new Worker('http://0.0.0.0:8686');

$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($_GET, $_POST);
    $connection->send("hello");
};

// Executar o worker
Worker::runAll();
```
