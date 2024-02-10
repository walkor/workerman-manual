# reloadable
## Descrição:
```php
bool Worker::$reloadable
```
Quando executado `php start.php reload`, um sinal de reload (SIGUSR1) será enviado para todos os processos filhos.

Após receber o sinal de reload, o processo filho irá automaticamente encerrar e o processo principal irá iniciar um novo processo, geralmente usado para atualizar o código do negócio.

Quando a propriedade $reloadable do processo é false, receber o sinal de reload acionará apenas o evento [onWorkerReload](on-worker-reload.md), e não reiniciará o processo atual.

Por exemplo, no modelo Gateway/Worker, o processo gateway é responsável por manter a conexão do cliente, enquanto o processo worker lida com as solicitações. Configurar a propriedade reloadable do processo gateway como false permite atualizar o código do negócio sem desconectar os clientes durante o reload.


## Exemplo

```php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// Define se este exemplo será reiniciado após receber um sinal de reload
$worker->reloadable = false;
$worker->onWorkerStart = function($worker)
{
    echo "Iniciando o Worker...\n";
};
// Executar o worker
Worker::runAll();
```
