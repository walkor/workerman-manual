# stdoutFile
## Descrição:
```php
static string Worker::$stdoutFile
```

Este é um atributo estático global. Se o processo for iniciado em modo daemon (`-d`), todas as saídas para o terminal (como echo, var_dump, etc.) serão redirecionadas para o arquivo especificado em stdoutFile.

Se não for configurado e for executado em modo daemon, todas as saídas do terminal serão redirecionadas para `/dev/null` (ou seja, descartando todas as saídas por padrão).

> Observação: `/dev/null` é um arquivo especial no Linux que na verdade é um buraco negro, onde todos os dados escritos nele são descartados. Se não quiser descartar a saída, você pode configurar `Worker::$stdoutFile` como o caminho de um arquivo normal.

> Observação: Este atributo deve ser configurado antes de executar ```Worker::runAll();```. Este recurso não é suportado no sistema Windows.

## Exemplo
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$daemonize = true;
// Todas as saídas de impressão são salvas no arquivo /tmp/stdout.log
Worker::$stdoutFile = '/tmp/stdout.log';
$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Início do worker\n";
};
// Executar o worker
Worker::runAll();
```
