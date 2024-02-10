# daemonize
## Descrição:
```php
static bool Worker::$daemonize
```

Este atributo é um atributo estático global que indica se o trabalho está sendo executado no modo daemon (processo de serviço). Se o comando de inicialização usar o parâmetro ```-d```, então esse atributo será automaticamente definido como verdadeiro. Também pode ser configurado manualmente no código.

Nota: Este atributo deve ser configurado antes de chamar  ```Worker::runAll();``` para ser eficaz. Este recurso não é suportado no sistema Windows.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start\n";
};
// Executar o worker
Worker::runAll();
```
