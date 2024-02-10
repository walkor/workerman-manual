# onClose
## 설명:
```php
callback Worker::$onClose
```

클라이언트 연결이 Workerman에서 끊어질 때 호출되는 콜백 함수입니다. 연결이 어떻게 끊어지든 간에 `onClose`가 호출됩니다. 각 연결은 한 번만 `onClose`를 호출합니다.

주의: 대상이 네트워크 장애 또는 전원 차단과 같은 극단적인 상황으로 연결이 끊어진 경우에는 Workerman에게 즉시 TCP FIN 패킷을 보낼 수 없기 때문에 연결이 이미 끊어졌음을 즉시 알 수 없으며, 따라서 `onClose`를 즉시 호출할 수 없습니다. 이러한 경우 애플리케이션 레벨의 하트비트를 사용하여 해결해야 합니다. Workerman에서의 연결 하트비트 구현은 [여기](../faq/heartbeat.md)에서 확인할 수 있습니다. GatewayWorker 프레임워크를 사용하는 경우에는 GatewayWorker 프레임워크의 하트비트 메커니즘을 직접 사용하면 됩니다. 자세한 내용은 [여기](https://doc2.workerman.net/heartbeat.html)를 참조하십시오.

UDP는 비연결형이므로 UDP를 사용할 때는 `onConnect` 콜백이 호출되지 않으며 `onClose` 콜백도 호출되지 않습니다.

## 콜백 함수의 매개변수

``` $connection ```

연결 객체, 즉 [TcpConnection 인스턴스](../tcp-connection.md), 클라이언트 연결의 작업에 사용됩니다. 예를 들어 [데이터를 보내기](../tcp-connection/send.md), [연결을 닫기](../tcp-connection/close.md) 등입니다.

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// worker 실행
Worker::runAll();
```

팁: 콜백으로 익명 함수를 사용하는 대신 [이 곳](../faq/callback_methods.md)에서 다른 콜백 작성 방법을 참고할 수 있습니다.
