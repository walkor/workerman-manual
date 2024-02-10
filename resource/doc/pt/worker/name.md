# nome

## Descrição:
```php
string Worker::$name
```

Define o nome da instância atual do Worker para facilitar a identificação do processo ao executar o comando de status. Se não configurado, o padrão é "none".

## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Define o nome da instância
$worker->name = 'MeuWorkerWebsocket';
$worker->onWorkerStart = function($worker)
{
    echo "Iniciando o worker...\n";
};
// Executa o worker
Worker::runAll();
```
