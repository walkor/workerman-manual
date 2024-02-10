# usuário

## Descrição:
```php
string Worker::$user
```

Define em qual usuário o atual Worker deve ser executado. Esta propriedade só terá efeito se o usuário atual for root. Se não for definida, será executada com o usuário atual. 

É recomendado definir `$user` como um usuário com permissões mais baixas, como www-data, apache, nobody, etc.

Observação: Esta propriedade deve ser definida antes de executar ```Worker::runAll();```. Este recurso não é suportado no sistema Windows.

## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Define o usuário de execução da instância
$worker->user = 'www-data';
$worker->onWorkerStart = function($worker)
{
    echo "Iniciando o worker...\n";
};
// Executa o worker
Worker::runAll();
```
