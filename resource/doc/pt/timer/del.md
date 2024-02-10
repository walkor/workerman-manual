# del
```php
boolean \Workerman\Timer::del(int $timer_id)
```
Eliminar um temporizador específico

### Parâmetros
 ``` timer_id ```

O ID do temporizador, ou seja, um número inteiro retornado pela interface add

### Valor de retorno
boolean

### Exemplo
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Inicia vários processos para executar tarefas agendadas, observe problemas de concorrência com vários processos
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Executa a cada 2 segundos
    $timer_id = Timer::add(2, function()
    {
        echo "Executando tarefa\n";
    });
    // Executa uma tarefa única após 20 segundos, excluindo a tarefa agendada de 2 segundos
    Timer::add(20, function($timer_id)
    {
        Timer::del($timer_id);
    }, array($timer_id), false);
};

// Executa o worker
Worker::runAll();
```

### Exemplo (excluir o temporizador atual dentro da chamada de retorno do temporizador)
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
$task->onWorkerStart = function(Worker $task)
{
    // Observe que ao utilizar o ID do temporizador atual na chamada de retorno, é necessário importá-lo como uma referência(&)
    $timer_id = Timer::add(1, function() use (&$timer_id)
    {
        static $i = 0;
        echo $i++."\n";
        // Exclui o temporizador após 10 execuções
        if($i === 10)
        {
            Timer::del($timer_id);
        }
    });
};

// Executa o worker
Worker::runAll();
```
