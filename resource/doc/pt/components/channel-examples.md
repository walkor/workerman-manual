# Exemplo 1
**```(Exige Workerman versão >=3.3.0)```**

Sistema de transmissão baseado em múltiplos processos (cluster distribuído) com base no Worker, enviando em cluster e transmissões em massa.

`start_channel.php`
Apenas um serviço start_channel pode ser implantado em todo o sistema. Suponha que esteja sendo executado em 192.168.1.1.
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/vendor/autoload.php';

// Inicializa um servidor de Canal
$channel_server = new Channel\Server('0.0.0.0', 2206);

Worker::runAll();
```

`start_ws.php`
Vários serviços start_ws podem ser implantados no sistema, operando em duas máquinas diferentes, 192.168.1.2 e 192.168.1.3.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Servidor websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->count=2;
$worker->name = 'pusher';
$worker->onWorkerStart = function($worker)
{
    // Conectar o cliente do Canal ao servidor do Canal
    Channel\Client::connect('192.168.1.1', 2206);
    // Usa o ID do próprio processo como nome do evento
    $event_name = $worker->id;
    // Assina o evento worker->id e registra a função de tratamento de eventos
    Channel\Client::on($event_name, function($event_data)use($worker){
        $to_connection_id = $event_data['to_connection_id'];
        $message = $event_data['content'];
        if(!isset($worker->connections[$to_connection_id]))
        {
            echo "conexão não existe\n";
            return;
        }
        $to_connection = $worker->connections[$to_connection_id];
        $to_connection->send($message);
    });

    // Assina o evento de transmissão em massa
    $event_name = 'broadcast';
    // Ao receber o evento de transmissão em massa, envia os dados de transmissão para todas as conexões de cliente dentro do próprio processo
    Channel\Client::on($event_name, function($event_data)use($worker){
        $message = $event_data['content'];
        foreach($worker->connections as $connection)
        {
            $connection->send($message);
        }
    });
};

$worker->onConnect = function(TcpConnection $connection)use($worker)
{
    $msg = "ID do Worker:{$worker->id} ID da Conexão:{$connection->id} conectada\n";
    echo $msg;
    $connection->send($msg);
};
Worker::runAll();
```

`start_http.php`
Vários serviços start_ws podem ser implantados no sistema, operando em duas máquinas diferentes, 192.168.1.4 e 192.168.1.5.
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Lida com solicitações http, enviando dados para qualquer cliente, requer o envio de ID do worker e ID de conexão
$http_worker = new Worker('http://0.0.0.0:4237');
$http_worker->name = 'publisher';
$http_worker->onWorkerStart = function()
{
    Channel\Client::connect('192.168.1.1', 2206);
};
$http_worker->onMessage = function(TcpConnection $connection, $request)
{
    // Compatível com workerman 4.x
    if (!is_array($request)) {
            $_GET = $request->get();
   }
    $connection->send('ok');
    if(empty($_GET['content'])) return;
    // Enviar dados para um processo de worker ou conexão específica
    if(isset($_GET['to_worker_id']) && isset($_GET['to_connection_id']))
    {
        $event_name = $_GET['to_worker_id'];
        $to_connection_id = $_GET['to_connection_id'];
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'to_connection_id' => $to_connection_id,
           'content'          => $content
        ));
    }
    // Transmissão em massa global
    else
    {
        $event_name = 'broadcast';
        $content = $_GET['content'];
        Channel\Client::publish($event_name, array(
           'content'          => $content
        ));
    }
};

Worker::runAll();
```

## Teste
1. Execute os serviços em cada servidor.
2. Conecte-se com o cliente ao servidor.
Abra o navegador Chrome, pressione F12 para abrir o console de depuração e insira (ou coloque o código abaixo em uma página HTML e execute com JS)

```javascript
// Também pode se conectar a ws://192.168.1.3:4236
ws = new WebSocket("ws://192.168.1.2:4236");
ws.onmessage = function(e) {
    alert("Mensagem recebida do servidor: " + e.data);
};
```

3. Envie através da chamada da interface HTTP
Acesse a URL `http://192.168.1.4:4237/?content={$content}`  ou  `http://192.168.1.5:4237/?content={$content}` para enviar dados `{$content}` a todas as conexões de clientes.

Acesse a URL `http://192.168.1.4:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}` ou `http://192.168.1.5:4237/?to_worker_id={$worker_id}&to_connection_id={$connection_id}&content={$content}` para enviar dados `{$content}` para uma conexão específica dentro de um processo de worker.

Nota: Ao testar, substitua `{$worker_id}`, `{$connection_id}` e `{$content}` pelos valores reais.
