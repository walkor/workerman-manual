# onError
## 설명:
```php
callback Worker::$onError
```

클라이언트 연결에 오류가 발생했을 때 트리거됩니다.

현재 오류 유형은 다음과 같습니다.

1. Client::send 호출로 인한 클라이언트 연결이 끊겨서 실패한 경우 (이어서 onClose 콜백이 트리거됨) ``` (code:WORKERMAN_SEND_FAIL msg:client closed)```

2. onBufferFull을 트리거한 후(전송 버퍼가 가득 찬 상태), 여전히 Connection::send를 호출하고 전송 버퍼가 여전히 가득 찬 상태로 인해 실패한 경우 (onClose 콜백은 트리거되지 않음) ```(code:WORKERMAN_SEND_FAIL msg:send buffer full and drop package)```

3. AsyncTcpConnection 비동기 연결이 실패한 경우 (이어서 onClose 콜백이 트리거됨) ```(code:WORKERMAN_CONNECT_FAIL msg:stream_socket_client가 반환한 오류 메시지)```

## 콜백 함수 매개변수

 ``` $connection ```

연결 객체, 즉 [TcpConnection 인스턴스](../tcp-connection.md)로, 클라이언트 연결을 조작하는 데 사용됩니다. 예를 들어 [데이터 전송](../tcp-connection/send.md), [연결 닫기](../tcp-connection/close.md) 등

 ``` $code ```

오류 코드

 ``` $msg ```

오류 메시지


## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onError = function(TcpConnection $connection, $code, $msg)
{
    echo "error $code $msg\n";
};
// worker 실행
Worker::runAll();
```

팁: 익명 함수 외에도 [여기](../faq/callback_methods.md)에서 다른 콜백 작성 방법을 확인할 수 있습니다.
