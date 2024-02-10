# id
Requer que ```（workerman >= 3.2.1）```

## Descrição:
```php
int Worker::$id
```

O número de identificação do processo worker atual, variando de ```0``` a ```$worker->count-1```.

Esta propriedade é muito útil para distinguir os processos worker. Por exemplo, se uma instância de worker tem vários processos e o desenvolvedor deseja configurar um temporizador em apenas um desses processos, ele pode fazer isso identificando o número do processo e configurando o temporizador apenas nesse número de identificação do worker (veja o exemplo).

## Observação:

Após a reinicialização do processo, o valor do número de identificação não será alterado.

A atribuição do número de identificação do processo é baseada em cada instância do worker. Cada instância do worker começa a numerar seus próprios processos a partir de 0, então pode haver repetição de números de identificação entre as instâncias do worker, mas não haverá repetição de números de identificação de processo dentro de uma única instância do worker. Por exemplo, no exemplo abaixo:

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// A instância do worker 1 tem 4 processos, e os números de identificação dos processos serão 0, 1, 2, 3
$worker1 = new Worker('tcp://0.0.0.0:8585');
// Configurar para iniciar 4 processos
$worker1->count = 4;
// Após o início de cada processo, imprime o número de identificação do processo atual ou seja, $worker1->id
$worker1->onWorkerStart = function($worker1)
{
    echo "worker1->id={$worker1->id}\n";
};

// A instância do worker 2 tem 2 processos, e os números de identificação dos processos serão 0, 1
$worker2 = new Worker('tcp://0.0.0.0:8686');
// Configurar para iniciar 2 processos
$worker2->count = 2;
// Após o início de cada processo, imprime o número de identificação do processo atual ou seja, $worker2->id
$worker2->onWorkerStart = function($worker2)
{
    echo "worker2->id={$worker2->id}\n";
};

// Executar worker
Worker::runAll();
```
A saída será semelhante a:
```
worker1->id=0
worker1->id=1
worker1->id=2
worker1->id=3
worker2->id=0
worker2->id=1
```

Observação: Devido ao fato de o sistema Windows não suportar a configuração do número de processos count, o número de identificação será apenas 0. Além disso, no sistema Windows, não é possível executar dois Workers de escuta a partir do mesmo arquivo, então este exemplo não funcionará no sistema Windows.

## Exemplo
Uma instância do worker tem 4 processos e configura um temporizador apenas no processo com o número de identificação 0.

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('tcp://0.0.0.0:8585');
$worker->count = 4;
$worker->onWorkerStart = function($worker)
{
    // Configura um temporizador apenas no processo com o número de identificação 0, os outros processos com os números 1, 2, 3 não terão temporizador configurado
    if($worker->id === 0)
    {
        Timer::add(1, function(){
            echo "4 processos worker, configura um temporizador apenas no processo 0\n";
        });
    }
};
// Executar worker
Worker::runAll();
```
