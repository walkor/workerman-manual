```php
# on
**``` (Workerman version >= 3.3.0 is required) ```**
```php
void \Channel\Client::on(string $event_name, callback $callback_function)
```
Subscreva o evento ```$event_name``` e registre a função de retorno de chamada ```$callback_function``` quando o evento ocorrer.

## Parâmetros da função de retorno de chamada

 ``` $event_name ```

O nome do evento subscrito, pode ser qualquer string.

 ``` $callback_function ```

A função de retorno de chamada acionada quando o evento ocorre. O protótipo da função é ```callback_function(mixed $event_data)```. ```$event_data``` é os dados do evento passados quando o evento é publicado.

Observação:

Se duas funções de retorno de chamada forem registradas para o mesmo evento, a segunda substituirá a primeira.


## Exemplo
Worker multi-process (multi-servidor), um cliente envia uma mensagem e transmite para todos os clientes.


start.php
```php
<?php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// Inicializa um servidor Channel
$channel_server = new Channel\Server('0.0.0.0', 2206);

// Servidor websocket
$worker = new Worker('websocket://0.0.0.0:4236');
$worker->name = 'websocket';
$worker->count = 6;
// Quando cada processo worker é iniciado
$worker->onWorkerStart = function($worker)
{
    // Conecta o cliente Channel ao servidor Channel
    Channel\Client::connect('127.0.0.1', 2206);
    // Subscreve o evento broadcast e registra a função de retorno de chamada do evento
    Channel\Client::on('broadcast', function($event_data)use($worker){
        // Transmite uma mensagem para todos os clientes do processo worker atual
        foreach($worker->connections as $connection)
        {
            $connection->send($event_data);
        }
    });
};

$worker->onMessage = function(TcpConnection $connection, $data)
{
   // Usa os dados enviados pelo cliente como dados do evento
   $event_data = $data;
   // Publica o evento broadcast para todos os processos worker
   \Channel\Client::publish('broadcast', $event_data);
};

Worker::runAll();
```

**Teste**

Abra o navegador Chrome, pressione F12 para abrir o console de depuração e, na guia Console, insira (ou insira o código a seguir em uma página HTML e execute com JS)

Conexão que recebe mensagens
```javascript
// Altere 127.0.0.1 para o IP real onde o Workerman está localizado
ws = new WebSocket("ws://127.0.0.1:4236");
ws.onmessage = function(e) {
    alert("Mensagem recebida do servidor: " + e.data);
};
```

Transmissão de mensagens
```javascript
ws.send('olá mundo');
```
