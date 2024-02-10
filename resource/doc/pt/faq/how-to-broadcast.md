# Como transmitir dados em broadcast (envio em massa)

## Exemplo (broadcast programado)

```php
use Workerman\Worker;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Neste exemplo, o número de processos deve ser 1
$worker->count = 1;
// Ao iniciar o processo, define um temporizador para enviar dados para todas as conexões dos clientes periodicamente
$worker->onWorkerStart = function($worker)
{
    // Envia a cada 10 segundos
    Timer::add(10, function() use ($worker)
    {
        // Percorre todas as conexões de clientes atuais do processo e envia a hora atual do servidor
        foreach ($worker->connections as $connection)
        {
            $connection->send(time());
        }
    });
};
// Executa o worker
Worker::runAll();
```

## Exemplo (chat em grupo)

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:2020');
// Neste exemplo, o número de processos deve ser 1
$worker->count = 1;
// Quando o cliente envia uma mensagem, broadcast para os outros usuários
$worker->onMessage = function(TcpConnection $connection, $message) use ($worker)
{
    foreach ($worker->connections as $connection)
    {
        $connection->send($message);
    }
};
// Executa o worker
Worker::runAll();
```

## Explicação:
**Único processo:**
Os exemplos acima só funcionam com **um único processo** (```$worker->count=1```), pois com vários processos, diferentes clientes podem ser conectados a processos diferentes, e as conexões dos clientes entre os processos são isoladas, não podendo se comunicar diretamente. Em outras palavras, o processo A não pode operar **diretamente** o objeto de conexão do cliente do processo B para enviar dados. (Para fazer isso, é necessário comunicação entre processos, como o uso do componente Channel, por exemplo [Exemplo - Envio em cluster](../components/channel-examples.md), [Exemplo - Envio em grupo](../components/channel-examples2.md)).

**Recomenda-se o uso do GatewayWorker:**
A estrutura GatewayWoker, desenvolvida com base no Workerman, oferece um mecanismo de push mais conveniente, incluindo multicast, broadcast, etc. Pode ser configurada com vários processos e até mesmo ser implantada em vários servidores. Se precisar enviar dados para os clientes, é recomendável utilizar a estrutura GatewayWorker.

Endereço do manual do GatewayWorker: https://doc2.workerman.net
