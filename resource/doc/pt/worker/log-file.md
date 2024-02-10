# logFile
## Descrição:
```php
static string Worker::$logFile
```

Usado para especificar a localização do arquivo de log do workerman.

Este arquivo registra os logs relacionados ao próprio workerman, incluindo inicialização, parada, etc.

Se não for definido, o nome do arquivo padrão será workerman.log, e a localização do arquivo será no diretório pai do Workerman.

**Nota:**

Este arquivo de log registra apenas os logs relacionados à inicialização, parada, etc., do workerman e não inclui nenhum log comercial.

Os logs do negócio podem ser implementados usando funções como [file_put_contents](https://php.net/manual/pt_BR/function.file-put-contents.php) ou [error_log](https://php.net/manual/pt_BR/function.error-log.php).

## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$logFile = '/tmp/workerman.log';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Início do worker";
};
// Executar o worker
Worker::runAll();
```
