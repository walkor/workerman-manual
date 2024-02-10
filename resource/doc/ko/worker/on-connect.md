# onConnect
## 설명:
```php
callback Worker::$onConnect
```

클라이언트가 Workerman과 연결을 수립할 때(TCP 3-way handshake 완료 후) 트리거되는 콜백 함수이다. 각 연결은 `onConnect` 콜백을 한 번만 트리거한다.

주의: onConnect 이벤트는 클라이언트가 Workerman과 TCP 3-way handshake를 완료했음을 의미할 뿐, 이 당시 클라이언트는 어떠한 데이터도 보내지 않았다. 따라서 `onConnect` 이벤트 내에서는 상대측이 누구인지를 확인할 방법이 없다. 누가 상대측인지를 알고 싶다면 클라이언트가 인증 데이터를 보내야 하며, 예를 들어 토큰이나 사용자 이름 및 암호와 같은 인증 데이터를 클라이언트가 전송한 후에는 [onMessage 콜백](on-message.md)에서 인증을 수행해야 한다.

UDP는 연결이 없기 때문에 UDP를 사용할 때는 onConnect 콜백이 트리거되지 않으며, 마찬가지로 onClose 콜백도 트리거되지 않는다.

## 콜백 함수의 매개변수
``` $connection ```

연결 객체, 즉 [TcpConnection 인스턴스](../tcp-connection.md)로, 클라이언트 연결을 조작하는 데 사용된다. 이는 [데이터 전송](../tcp-connection/send.md), [연결 닫기](../tcp-connection/close.md) 등의 기능을 수행한다.

## 예시
```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};
// worker 실행
Worker::runAll();
```

참고: 콜백으로 익명 함수를 사용하는 대신 다른 콜백 작성 방법에 대해서는 [여기를 참조](../faq/callback_methods.md)할 수 있다.
