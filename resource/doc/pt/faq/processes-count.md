# Quantos processos devem ser iniciados

## Como definir o número de processos
O número de processos é determinado pelo atributo ```count``` (o sistema Windows não suporta a configuração do número de processos), como no código abaixo:
```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$http_worker = new Worker("http://0.0.0.0:2345");

// ## Iniciar 4 processos para fornecer serviços externos ##
$http_worker->count = 4;

...
```

## Considerações ao definir o número de processos
1. Número de núcleos da CPU
2. Tamanho da memória
3. Tipo de carga de trabalho: intensiva em E/S ou intensiva em CPU

## Princípios para definir o número de processos

1. A soma da memória ocupada por cada processo deve ser inferior à memória total (geralmente, cada processo de negócio ocupa cerca de 40 MB)
2. Se for intensivo em E/S, ou seja, se envolver operações de E/S **bloqueantes**, como o acesso usual ao MySQL, Redis e etc., o número de processos pode ser aumentado, como configurar três vezes o número de núcleos da CPU. Se houver muitas operações bloqueantes no negócio, o número de processos pode ser aumentado novamente, por exemplo, até 8 vezes o número de núcleos da CPU ou ainda mais. Note que E/S **não bloqueantes** são consideradas intensivas em CPU, e não em E/S.
3. Se for intensivo em CPU, ou seja, se não houver operações de E/S **bloqueantes** no negócio, como leitura assíncrona de recursos de rede, desde que o processo não seja bloqueado pelo código do negócio, o número de processos pode ser configurado igual ao número de núcleos da CPU.

## Valores de referência para definir o número de processos
Se o código do negócio for intensivo em E/S, o número de processos pode ser definido de acordo com o grau de intensidade de E/S, por exemplo, de 3 a 8 vezes o número de núcleos da CPU.

Se o código do negócio for intensivo em CPU, o número de processos pode ser configurado igual ao número de núcleos da CPU.

## Observação
O IO do próprio WorkerMan é não bloqueante, por exemplo, o```Connection->send```, todos são operações não bloqueantes, o que corresponde a operações intensivas em CPU. Se não estiver claro sobre qual tipo o seu negócio se enquadra, pode-se configurar o número de processos para cerca de 3 vezes o número de núcleos da CPU.
Além disso, mais processos nem sempre significam melhor desempenho. Se houver muitos processos em execução, o custo de troca de contexto aumentará, o que terá um certo impacto no desempenho.
