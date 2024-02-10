# SSE
**Este recurso requer workerman>=4.0.0**

SSE, ou Server-sent Events, é uma técnica de push do servidor. Sua essência é que, após o cliente enviar uma solicitação HTTP com o cabeçalho `Accept: text/event-stream`, a conexão não é fechada, e o servidor pode continuar enviando dados para o cliente nesta conexão.

A diferença entre SSE e websocket é:
*   SSE só permite o servidor enviar dados para o cliente; o WebSocket permite comunicação bidirecional.
*   SSE suporta reconexão automática por padrão; o WebSocket requer implementação própria.
*   SSE só pode transmitir texto UTF-8, e os dados binários precisam ser codificados em UTF-8 antes de serem enviados; o WebSocket suporta por padrão a transmissão de dados em texto UTF-8 e binários.
*   SSE possui tipos de mensagem embutidos; o WebSocket requer implementação própria.

### Exemplo
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function(TcpConnection $connection, Request $request)
{
    // Se o cabeçalho Accept for text/event-stream, é uma solicitação SSE
    if ($request->header('accept') === 'text/event-stream') {
        // Primeiro envia uma resposta com o cabeçalho Content-Type: text/event-stream
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Envie dados para o cliente em intervalos regulares
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // Deletar o temporizador quando a conexão for fechada, evitando vazamento de memória devido ao acúmulo contínuo de temporizadores
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Envia um evento 'message' com os dados 'hello' e o ID da mensagem como 1
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Executar worker
Worker::runAll();
```

Código javascript do cliente
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // Saída: hello
}, false);
```
