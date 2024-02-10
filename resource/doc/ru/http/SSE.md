# SSE 
**Эта функция требует workerman>=4.0.0**

SSE, или Server-sent Events, это технология серверной отправки. Ее суть заключается в том, что клиент отправляет HTTP-запрос с заголовком `Accept: text/event-stream`, после чего соединение не закрывается, и сервер может продолжать отправлять данные клиенту через это соединение.

Его отличие от WebSocket заключается в следующем:
* SSE позволяет только серверу отправлять данные клиенту; WebSocket позволяет обоюдное взаимодействие.
* SSE по умолчанию поддерживает переподключение после разрыва соединения; для WebSocket это нужно реализовывать самостоятельно.
* SSE может передавать только текст в формате UTF-8, бинарные данные должны быть закодированы в UTF-8 перед передачей; WebSocket по умолчанию поддерживает передачу как текста в формате UTF-8, так и бинарных данных.
* SSE имеет встроенные типы сообщений; для WebSocket это нужно реализовывать самостоятельно.

### Пример
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
    // Если заголовок Accept указывает на text/event-stream, значит это запрос SSE
    if ($request->header('accept') === 'text/event-stream') {
        // Сначала отправляем ответ с заголовком Content-Type: text/event-stream
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // Периодически отправляем данные клиенту
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // При закрытии соединения нужно удалить таймер, чтобы избежать накопления таймеров и утечек памяти
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // Отправляем событие message, с данными "hello", id сообщения можно не указывать
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Запуск worker
Worker::runAll();
```

JavaScript-код клиента
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // выводит hello
}, false);
```
