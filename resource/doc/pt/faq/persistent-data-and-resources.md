# Persistência de Objetos e Recursos

Na tradicional desenvolvimento web, os objetos, dados e recursos criados pelo PHP são liberados após a conclusão da solicitação, tornando difícil alcançar a persistência. No entanto, com o WorkerMan, isso pode ser facilmente alcançado.

No WorkerMan, se desejar manter permanentemente certos recursos de dados na memória, é possível colocar os recursos em variáveis globais ou em membros estáticos de uma classe.

Por exemplo, no código abaixo:

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Variável global para armazenar o número de conexões de clientes no processo atual
$connection_count = 0;

$worker = new Worker('tcp://0.0.0.0:1236');

$worker->onConnect = function(TcpConnection $connection)
{
    // Ao receber uma nova conexão de cliente, o número de conexões é incrementado em 1
    global $connection_count;
    ++$connection_count;
    echo "agora connection_count=$connection_count\n";
};

$worker->onClose = function(TcpConnection $connection)
{
    // Ao fechar a conexão do cliente, o número de conexões é decrementado em 1
    global $connection_count;
    $connection_count--;
    echo "agora connection_count=$connection_count\n";
};
```

## Consulte o escopo de variáveis PHP em:
https://php.net/manual/pt_BR/language.variables.scope.php
