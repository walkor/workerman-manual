# count

## Descrição:
```php
int Worker::$count
```

Define quantos processos a instância atual do Worker irá iniciar, por padrão é 1, se não especificado.

Para saber como definir o número de processos, consulte [**aqui**](../faq/processes-count.md).

Observação: Esta propriedade deve ser definida antes de chamar ```Worker::runAll();``` para ser efetiva. Este recurso não é suportado no sistema Windows.

## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Inicia 8 processos e escuta na porta 8484, fornecendo serviço através do protocolo websocket
$worker->count = 8;
$worker->onWorkerStart = function($worker)
{
    echo "Iniciando worker...\n";
};
// Executa o worker
Worker::runAll();
```
