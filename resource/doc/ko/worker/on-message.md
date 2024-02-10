# onMessage
## 설명:
```php
callback Worker::$onMessage
```

클라이언트가 연결을 통해 데이터를 보낼 때(Workerman이 데이터를 받을 때) 발생하는 콜백 함수

## 콜백 함수의 매개변수

```$connection```

연결 객체, 즉 [TcpConnection 인스턴스](../tcp-connection.md), 클라이언트 연결을 조작하는 데 사용됨, 예를 들어 [데이터 보내기](../tcp-connection/send.md), [연결 닫기](../tcp-connection/close.md) 등

```$data```

클라이언트 연결에서 전송된 데이터, 만약 Worker가 프로토콜을 지정했다면, $data는 해당 프로토콜 decode(해독)된 데이터입니다. 데이터 유형은 프로토콜 `decode()` 구현에 따라 달라지며, `websocket`, `text`, `frame`은 문자열이고, HTTP 프로토콜은 [`Workerman\Protocols\Http\Request`](../http/request.md) 객체입니다.

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('수신 성공');
};
// worker 실행
Worker::runAll();
```

참고: 콜백으로 익명 함수를 사용하는 것 외에도 [여기를 참고하세요](../faq/callback_methods.md) 다른 콜백 작성 방법을 사용할 수 있습니다.
