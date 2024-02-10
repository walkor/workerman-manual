# onBufferFull
## 설명:
```php
callback Worker::$onBufferFull
```

각 연결에는 별도의 응용 프로그램 수준 송신 버퍼가 있습니다. 클라이언트의 수신 속도가 서버의 송신 속도보다 느린 경우 데이터는 응용 프로그램 수준 버퍼에 보관되며, 버퍼가 가득 차면 onBufferFull 콜백이 트리거됩니다.

버퍼의 최대 크기는 [TcpConnection::$maxSendBufferSize](../tcp-connection/max-send-buffer-size.md)이며, 기본값은 1MB이며 현재 연결에 동적으로 버퍼 크기를 설정할 수 있습니다. 다음과 같이 설정할 수 있습니다.
```php
// 현재 연결의 송신 버퍼 크기를 설정합니다. 단위는 바이트입니다.
$connection->maxSendBufferSize = 102400;
```
또한 [TcpConnection::$defaultMaxSendBufferSize](../tcp-connection/default-max-send-buffer-size.md)를 사용하여 모든 연결의 기본 버퍼 크기를 설정할 수 있습니다. 다음은 예시입니다.
```php
use Workerman\Connection\TcpConnection;
// 모든 연결의 기본 응용 프로그램 송신 버퍼 크기를 설정합니다. 단위는 바이트입니다.
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;
```

이 콜백은 Connection::send 호출 후 즉시 트리거될 수 있습니다. 예를 들어 큰 데이터를 전송하거나 대량의 데이터를 연속해서 빠르게 대상에 전송하게 될 경우, 네트워크 등의 이유로 데이터가 해당 연결의 송신 버퍼에 대량으로 쌓일 경우 초과하면 onBufferFull 이벤트가 발생합니다.

onBufferFull 이벤트가 발생할 때 개발자는 대상에 데이터를 전송하는 것을 중지하고 송신 버퍼의 데이터가 전송될 때까지 기다리는 조치를 취해야 합니다(onBufferDrain 이벤트). 

Connection::send(`$A`)를 호출하여 onBufferFull 이벤트가 발생하면, 이번 send의 데이터`$A`가 얼마가 큰지에 상관없이`TcpConnection::$maxSendBufferSize`를 초과하더라도 이번에 전송할 데이터는 여전히 송신 버퍼에 넣습니다. 즉, 송신 버퍼에 실제로 넣은 데이터는`TcpConnection::$maxSendBufferSize`보다 훨씬 크게 될 수 있습니다. 따라서 송신 버퍼의 데이터가 이미`TcpConnection::$maxSendBufferSize`보다 큰 경우에도 계속해서 Connection::send(`$B`) 데이터를 전송하면, 이번에 보낸`$B` 데이터가 송신 버퍼에 넣는 대신 버려지고`onError` 콜백이 트리거됩니다.

즉, 송신 버퍼가 아직 가득 차 있지 않은 경우에도 공간이 하나의 바이트만 있더라도 Connection::send(```$A```)를 호출하면```$A```를 송신 버퍼에 넣게 됩니다. 송신 버퍼에 넣은 후에 송신 버퍼의 크기가`TcpConnection::$maxSendBufferSize` 제한을 초과하면 onBufferFull 콜백이 트리거됩니다.


## 콜백 함수의 매개 변수

``` $connection ```

연결 개체, 즉 [TcpConnection 인스턴스](../tcp-connection.md)이며, 클라이언트 연결을 조작하는 데 사용됩니다. 즉, [데이터 전송](../tcp-connection/send.md), [연결 종료](../tcp-connection/close.md) 등이 있습니다.


## 예제
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "bufferFull and do not send again\n";
};
// worker 실행
Worker::runAll();
```

참고: 익명 함수 외에도 [여기](../faq/callback_methods.md)에서 다른 콜백 작성 방법을 참조할 수 있습니다.


## 참고
onBufferDrain: 연결의 응용 프로그램 송신 버퍼의 데이터가 모두 전송된 경우 트리거됩니다.
