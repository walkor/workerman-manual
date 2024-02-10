# pidFile
## Descrição:
```php
static string Worker::$pidFile
```

É recomendável não definir esta propriedade, a menos que haja uma necessidade específica.

Esta propriedade é uma propriedade estática global usada para definir o caminho do arquivo pid para os processos do WorkerMan.

Essa configuração é útil para monitoramento, por exemplo, armazenar o arquivo pid do WorkerMan em um diretório fixo pode facilitar a leitura do arquivo pid por algum software de monitoramento para acompanhar o status do processo do WorkerMan.

Se não for definido, o WorkerMan irá gerar automaticamente um arquivo pid em um local paralelo ao diretório do Workerman (observe que, antes da versão 3.2.3 do workerman, o padrão era em ```sys_get_temp_dir()```) e, para evitar conflitos de pid ao iniciar várias instâncias do WorkerMan, o arquivo pid gerado pelo WorkerMan inclui o caminho do WorkerMan atual.

Observação: Esta propriedade deve ser definida antes de executar ```Worker::runAll();```. Este recurso não é suportado no sistema Windows.

## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

Worker::$pidFile = '/var/run/workerman.pid';

$worker = new Worker('text://0.0.0.0:8484');
$worker->onWorkerStart = function($worker)
{
    echo "Worker start";
};
// Executar worker
Worker::runAll();
```
