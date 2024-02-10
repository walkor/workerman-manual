# Como implementar tarefas assíncronas

**Pergunta:**

Como lidar de forma assíncrona com tarefas pesadas para evitar que a operação principal seja bloqueada por longos períodos de tempo. Por exemplo, se eu precisar enviar e-mails para 1000 usuários, esse processo é lento e pode bloquear o servidor por alguns segundos. Durante esse período, o bloqueio da operação principal pode afetar as solicitações subsequentes. Como posso enviar tarefas pesadas para serem processadas de forma assíncrona por outros processos?

**Resposta:**

Você pode estabelecer préviamente em um único servidor, em outro servidor ou até mesmo em um cluster de servidores, alguns processos de tarefas para lidar com tarefas pesadas. O número de processos de tarefas pode ser aumentado, por exemplo, 10 vezes a quantidade de CPUs. Em seguida, o chamador pode usar a AsyncTcpConnection para enviar os dados de forma assíncrona para esses processos de tarefa para serem processados assincronamente e obter os resultados de forma assíncrona.

Servidor de processos de tarefas:
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Trabalhador de tarefas, usando o protocolo Text
$task_worker = new Worker('Text://0.0.0.0:12345');
// O número de processos de tarefas pode ser aumentado conforme necessário
$task_worker->count = 100;
$task_worker->name = 'TaskWorker';
$task_worker->onMessage = function(TcpConnection $connection, $task_data)
{
     // Suponha que os dados recebidos sejam do tipo JSON
     $task_data = json_decode($task_data, true);
     // Processar os dados da tarefa correspondente a task_data.... Obter o resultado, vamos omitir isso...
     $task_result = ......
     // Enviar o resultado
     $connection->send(json_encode($task_result));
};
Worker::runAll();
```

Chamada no workerman:

```php
use Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Serviço websocket
$worker = new Worker('websocket://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $ws_connection, $message)
{
    // Estabelece uma conexão assíncrona com o serviço de tarefas remoto, o IP é o do serviço de tarefas remoto, se for o mesmo servidor, é 127.0.0.1, se for um cluster é o IP do LVS
    $task_connection = new AsyncTcpConnection('Text://127.0.0.1:12345');
    // Dados da tarefa e argumentos
    $task_data = array(
        'function' => 'send_mail',
        'args'       => array('from'=>'xxx', 'to'=>'xxx', 'contents'=>'xxx'),
    );
    // Envio de dados
    $task_connection->send(json_encode($task_data));
    // Recebimento assíncrono do resultado
    $task_connection->onMessage = function(AsyncTcpConnection $task_connection, $task_result)use($ws_connection)
    {
         // Resultado
         var_dump($task_result);
         // Após receber o resultado, lembre-se de fechar a conexão assíncrona
         $task_connection->close();
         // Notificar o cliente do websocket que a tarefa foi concluída
         $ws_connection->send('tarefa concluída');
    };
    // Executa a conexão assíncrona
    $task_connection->connect();
};

Worker::runAll();
```

Dessa forma, as tarefas pesadas são processadas por processos no mesmo servidor ou em outros servidores, e quando a tarefa é concluída, o resultado é recebido de forma assíncrona, garantindo que o processo de negócios não seja bloqueado.
