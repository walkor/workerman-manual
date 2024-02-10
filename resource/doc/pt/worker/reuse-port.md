# reusePort
> **Nota**
> Requer o Workerman >= 3.2.1 PHP >= 7.0. O sistema Windows e o macOS não suportam esse recurso.

## Descrição:

```php
bool Worker::$reusePort
```

Define se o atual worker vai abrir a reutilização da porta de escuta (opção SO_REUSEPORT do socket).

Ao ativar a reutilização da porta de escuta permite que vários processos não relacionados escutem a mesma porta, e o balanceamento de carga é feito pelo kernel do sistema, decidindo a qual processo encaminhar a conexão do socket, evitando o efeito de muitos golpes, e pode melhorar o desempenho de aplicativos de conexão curta com múltiplos processos.

**Nota:** Esse recurso requer a versão do PHP >= 7.0

## Exemplo 1

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->count = 4;
$worker->reusePort = true;
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('ok');
};
// Executa o worker
Worker::runAll();
```

## Exemplo 2: Ouvir várias portas (múltiplos protocolos) no Workerman
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('text://0.0.0.0:2015');
$worker->count = 4;
// Após a inicialização de cada processo, adiciona-se uma nova escuta no processo atual
$worker->onWorkerStart = function($worker)
{
    $inner_worker = new Worker('http://0.0.0.0:2016');
    /**
     * Vários processos escutando a mesma porta (o socket de escuta não é herdado pelo processo pai)
     * Requer a reutilização da porta, caso contrário, ocorrerá um erro de Address already in use
     */
    $inner_worker->reusePort = true;
    $inner_worker->onMessage = 'on_message';
    // Realiza a escuta
    $inner_worker->listen();
};

$worker->onMessage = 'on_message';

function on_message(TcpConnection $connection, $data)
{
    $connection->send("hello\n");
}

// Executa o worker
Worker::runAll();
```
