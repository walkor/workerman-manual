# SSE
**Bu özellik workerman>=4.0.0 gerektirir.**

SSE yani Server-sent Events, bir sunucu tarafından yayınlanan bir teknolojidir. Temelde, istemci `Accept: text/event-stream` başlığını taşıyan bir HTTP isteği gönderdikten sonra bağlantı kapanmaz ve sunucu bu bağlantı üzerinden sürekli olarak veri gönderebilir.

Websocket'ten farkı:
* SSE sadece sunucudan istemciye yönlendirme yapabilir; Websocket çift yönlü iletişim sağlayabilir.
* SSE varsayılan olarak bağlantı kopması durumunda yeniden bağlanmayı destekler; WebSocket'i kendiniz uygulamanız gerekir.
* SSE yalnızca UTF-8 metin iletimi yapabilir, ikili veriler UTF-8'e kodlanarak iletilmelidir; WebSocket varsayılan olarak UTF-8 ve ikili veri iletimini destekler.
* SSE kendi mesaj türlerini içerir; WebSocket'i kendiniz uygulamanız gerekir.

### Örnek
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
    // Eğer Accept başlığı text/event-stream ise, SSE isteği olduğunu belirtir
    if ($request->header('accept') === 'text/event-stream') {
        // İlk olarak, Content-Type: text/event-stream başlığıyla bir yanıt gönderilir
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // İstemciye düzenli olarak veri gönderme
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id){
            // Bağlantıyı kapattığımızda, sürekli birikmeyi önlemek için zamanlayıcıyı silmemiz gerekir
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // message olayını gönder, veri olarak hello, mesaj kimliği atlanabilir
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id'=>1]));
        });
        return;
    }
    $connection->send('ok');
};

// Worker'ı çalıştır
Worker::runAll();
```

İstemci JavaScript kodu
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function (event) {
  var data = event.data;
  console.log(data); // hello çıktısını verir
}, false);
```
