# onWorkerStart
## Descrição:
```php
callback Worker::$onWorkerStart
```

Define a função de retorno de chamada para quando o processo filho do Worker for iniciado, sendo executada cada vez que um processo filho for iniciado.

Observação: onWorkerStart é executado quando um processo filho é iniciado. Se multiplos processos filhos forem iniciados (```$worker->count > 1```), a função será executada ```$worker->count``` vezes no total.


## Parâmetros da função de retorno de chamada

``` $worker ```

O objeto Worker


## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onWorkerStart = function(Worker $worker)
{
    echo "Iniciando o worker...\n";
};
// Executar o worker
Worker::runAll();
```

Nota: Além de usar uma função anônima como retorno de chamada, você também pode [consultar aqui](../faq/callback_methods.md) para outras formas de retorno de chamada.
