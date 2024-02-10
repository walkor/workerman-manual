```php
int \Workerman\Timer::add(float $time_interval, callable $callback [,$args = array(), bool $persistent = true])
```
Executa uma função ou método de classe em intervalos regulares.

Aviso: Os temporizadores são executados no processo atual, não sendo criados novos processos ou threads para executar os temporizadores no workerman.

### Parâmetros
 ``` time_interval ```

Tempo em segundos entre cada execução, suporta números decimais, com precisão até 0.001, ou seja, em nível de milissegundos.

 ``` callback ```

Função de retorno. **Atenção: Se o retorno da função for um método de classe, o método deve ser público.**

 ``` args ```

Parâmetros da função de retorno, deve ser um array com os valores dos parâmetros.

 ``` persistent ```

Se é persistente, se deseja executar o temporizador apenas uma vez, passe false (tarefas que são executadas apenas uma vez serão automaticamente destruídas após a execução, sem a necessidade de chamar ```Timer::del()```). O padrão é true, ou seja, continuará executando regularmente.

### Valor de retorno
Retorna um inteiro que representa o ID do temporizador, que pode ser destruído chamando ```Timer::del($timerid)```.

### Exemplos

#### 1. Função de temporização anônima (fechamento)
```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// Inicia o trabalho em várias instâncias para executar tarefas com temporização (verifique se o negócio tem um problema de concorrência em várias instâncias)
$task->count = 1;
$task->onWorkerStart = function(Worker $task)
{
    // Executar a cada 2.5 segundos
    $time_interval = 2.5;
    Timer::add($time_interval, function()
    {
        echo "tarefa executada\n";
    });
};

// Executar o worker
Worker::runAll();
```
