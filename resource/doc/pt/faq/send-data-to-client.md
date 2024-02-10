# Como enviar dados para um cliente específico no WorkerMan
Ao utilizar o Worker como servidor, sem o GatewayWorker, como posso enviar mensagens para um usuário específico?

```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inicializa um container worker, escutando na porta 1234
$worker = new Worker('websocket://workerman.net:1234');
// ====O número de processos deve ser definido como 1====
$worker->count = 1;
// Adiciona um atributo para mapear o uid para a conexão (uid é o id do usuário ou o identificador único do cliente)
$worker->uidConnections = array();
// Função de callback quando um cliente envia uma mensagem
$worker->onMessage = function(TcpConnection $connection, $data)
{
    global $worker;
    // Verifica se o cliente atual já foi autenticado, isto é, se o uid foi definido
    if(!isset($connection->uid))
    {
       // Se não estiver autenticado, o primeiro pacote é tratado como uid (Para fins de demonstração, não é feita uma verificação real)
       $connection->uid = $data;
       /* Mapeia o uid para a conexão, o que permite encontrar a conexão facilmente através do uid e enviar dados direcionados a um uid específico */
       $worker->uidConnections[$connection->uid] = $connection;
       return $connection->send('Login realizado com sucesso, seu uid é ' . $connection->uid);
    }
    // Outra lógica para enviar a um uid específico, ou para fazer uma transmissão global
    // Assume-se que o formato da mensagem é uid:mensagem para enviar a mensagem para um uid específico
    // Se o uid for "all", é feita uma transmissão global
    list($recv_uid, $message) = explode(':', $data);
    // Transmissão global
    if($recv_uid == 'all')
    {
        broadcast($message);
    }
    // Envia para um uid específico
    else
    {
        sendMessageByUid($recv_uid, $message);
    }
};

// Quando um cliente se desconecta
$worker->onClose = function(TcpConnection $connection)
{
    global $worker;
    if(isset($connection->uid))
    {
        // Remove o mapeamento da conexão ao desconectar
        unset($worker->uidConnections[$connection->uid]);
    }
};

// Envia dados para todos os usuários autenticados
function broadcast($message)
{
   global $worker;
   foreach($worker->uidConnections as $connection)
   {
        $connection->send($message);
   }
}

// Envia dados para um uid específico
function sendMessageByUid($uid, $message)
{
    global $worker;
    if(isset($worker->uidConnections[$uid]))
    {
        $connection = $worker->uidConnections[$uid];
        $connection->send($message);
    }
}

// Executa todos os workers (apesar de atualmente apenas um estar definido)
Worker::runAll();
```
**Observação:**

O exemplo acima permite a transmissão direcionada por uid e, embora seja um único processo, é capaz de suportar cerca de 100.000 usuários online sem problemas.

Atenção: Esse exemplo permite apenas um único processo, ou seja, $worker->count deve ser 1. Para suportar vários processos ou um cluster de servidores, é necessário utilizar o componente Channel para comunicação entre os processos. O desenvolvimento também é bastante simples e pode ser consultado na seção [Exemplo de transmissão em cluster usando o componente Channel](../components/channel-examples.md).

**Se desejar enviar mensagens para clientes em outros sistemas, consulte a seção [Enviar em outros projetos](push-in-other-project.md)**
