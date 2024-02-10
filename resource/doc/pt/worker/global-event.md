# globalEvent

## Descrição:
```php
static Event Worker::$globalEvent
```

Este atributo é uma propriedade estática global que representa a instância global de eventloop, na qual é possível registrar eventos de leitura/escrita de descritores de arquivo ou eventos de sinal.

## Exemplo

```php
use Workerman\Worker;
use Workerman\Events\EventInterface;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();
$worker->onWorkerStart = function($worker)
{
    echo 'O PID é ' . posix_getpid() . "\n";
    // Quando o processo recebe o sinal SIGALRM, imprime algumas informações
    Worker::$globalEvent->add(SIGALRM, EventInterface::EV_SIGNAL, function()
    {
        echo "Recebeu o sinal SIGALRM\n";
    });
};
// Executar o worker
Worker::runAll();
```

## Teste
Depois que o Workerman é iniciado, o PID do processo atual (um número) será exibido. No terminal, execute o comando
```sh
kill -SIGALRM pid_do_processo
```
O servidor imprimirá
```sh
Recebeu o sinal SIGALRM
```
