# SSE
**이 기능은 workerman 버전 4.0.0 이상이 필요합니다.**

SSE는 Server-sent Events의 약자로, 서버에서 클라이언트로 데이터를 전송하는 기술입니다. 핵심은 클라이언트가 `Accept: text/event-stream` 헤더를 갖는 HTTP 요청을 보낸 후에 연결을 닫지 않고, 서버가 이 연결을 통해 지속적으로 데이터를 클라이언트에게 푸시할 수 있다는 것입니다.

웹소켓과의 차이점은 다음과 같습니다.
* SSE는 서버에서 클라이언트로만 푸시할 수 있지만, 웹소켓은 양방향 통신이 가능합니다.
* SSE는 기본적으로 연결이 끊겼을 때 재연결을 지원하지만, 웹소켓은 직접 구현해야 합니다.
* SSE는 UTF-8 텍스트만 전송할 수 있으며, 이진 데이터는 UTF-8로 인코딩해야 합니다. 웹소켓은 기본적으로 UTF-8 및 이진 데이터를 전송할 수 있습니다.
* SSE에는 메시지 유형이 내장되어 있지만, 웹소켓은 직접 구현해야 합니다.

### 예제
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
use Workerman\Protocols\Http\Request;
use Workerman\Protocols\Http\ServerSentEvents;
use Workerman\Protocols\Http\Response;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('http://0.0.0.0:8080');

$worker->onMessage = function (TcpConnection $connection, Request $request) {
    // Accept 헤더가 text/event-stream이면 SSE 요청이라고 간주
    if ($request->header('accept') === 'text/event-stream') {
        // 먼저 Content-Type: text/event-stream 헤더를 포함한 응답을 보냄
        $connection->send(new Response(200, ['Content-Type' => 'text/event-stream'], "\r\n"));
        // 정기적으로 클라이언트에 데이터를 푸시
        $timer_id = Timer::add(2, function () use ($connection, &$timer_id) {
            // 연결이 닫힌 경우 타이머를 삭제하여 메모리 누수를 방지
            if ($connection->getStatus() !== TcpConnection::STATUS_ESTABLISHED) {
                Timer::del($timer_id);
                return;
            }
            // message 이벤트를 보냄, 데이터는 hello이며, 메시지 ID는 전달하지 않아도 됨
            $connection->send(new ServerSentEvents(['event' => 'message', 'data' => 'hello', 'id' => 1]));
        });
        return;
    }
    $connection->send('ok');
};

// worker 실행
Worker::runAll();
```

클라이언트 JavaScript 코드
```js
var source = new EventSource('http://127.0.0.1:8080');
source.addEventListener('message', function(event) {
  var data = event.data;
  console.log(data); // hello를 출력
}, false);
```
