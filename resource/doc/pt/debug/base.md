# Depuração Básica

O Workerman possui dois modos de execução, o modo de depuração e o modo de execução do daemon.

Execute `php start.php start` para entrar no modo de depuração. Neste modo, as funções de impressão como `echo`, `var_dump` e `var_export` serão exibidas no terminal. Observe que ao executar `php start.php start`, todos os processos do Workerman serão encerrados quando o terminal for fechado.

Já o modo de execução do daemon pode ser iniciado com o comando `php start.php start -d`, o que representa o modo de execução formalizado para produção, sem ser afetado pelo fechamento do terminal.

Se desejar ver as saídas das funções de impressão, como `echo`, `var_dump` e `var_export`, ao executar no modo de daemon, é possível configurar a propriedade Worker::$stdoutFile, como ilustrado no exemplo a seguir:

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Redirecionar a saída do console para o arquivo especificado em Worker::$stdoutFile
Worker::$stdoutFile = '/tmp/stdout.log';

$http_worker = new Worker("http://0.0.0.0:2345");
$http_worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello world');
};

Worker::runAll();
```

Dessa forma, todas as saídas das funções de impressão, como `echo`, `var_dump` e `var_export`, serão gravadas no arquivo especificado em `Worker::$stdoutFile`. Observe que o caminho especificado em `Worker::$stdoutFile` deve ter permissão de escrita.
