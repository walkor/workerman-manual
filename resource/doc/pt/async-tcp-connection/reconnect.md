# Método reConnect
```php
void AsyncTcpConnection::reConnect(float $delay = 0)
```

``` (Requer Workerman versão >=3.3.5) ```

Reconectar. Geralmente chamado no retorno de chamada ```onClose```, implementa a reconexão após a desconexão.

Se a conexão for interrompida devido a problemas de rede ou reinicialização do serviço remoto, você pode chamar este método para reconectar.

### Parâmetros
``` $delay ```

Tempo de espera para realizar a reconexão. Medido em segundos, suporta valores decimais, podendo ser preciso até milissegundos.

Se não for passado nenhum valor ou o valor for 0, significa que a reconexão será imediata.

É melhor passar um valor para atrasar a reconexão, evitando altos custos de CPU devido a problemas de conexão do serviço remoto.

### Valor de retorno
Sem valor de retorno

### Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Se a conexão for fechada, reconectar em 1 segundo
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```

> **Observação**
> Após a reconexão bem-sucedida, o método onConnect de $con será chamado novamente (se estiver configurado). Às vezes, queremos que o método onConnect seja executado apenas uma vez e não durante a reconexão, consulte o exemplo abaixo:

```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker)
{
    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    $con->onConnect = function(AsyncTcpConnection $con) {
        static $is_first_connect = true;
        if (!$is_first_connect) return;
        $is_first_connect = false;
        $con->send('hello');
    };
    $con->onMessage = function(AsyncTcpConnection $con, $msg) {
        echo "recv $msg\n";
    };
    $con->onClose = function(AsyncTcpConnection $con) {
        // Se a conexão for fechada, reconectar em 1 segundo
        $con->reConnect(1);
    };
    $con->connect();
};

Worker::runAll();
```
