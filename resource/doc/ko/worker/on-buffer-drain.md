# onBufferDrain
## 설명:
```php
callback Worker::$onBufferDrain
```

각 연결은 개별 응용 계층 송신 버퍼를 가지고 있으며, 버퍼 크기는 ```TcpConnection::$maxSendBufferSize```에 따라 결정되며, 기본값은 1MB이며 수동으로 크기를 변경할 수 있으며, 변경한 이후 모든 연결에 적용됩니다.

이 콜백은 응용 계층 송신 버퍼 데이터가 모두 전송된 후에 트리거됩니다. 일반적으로 onBufferFull과 함께 사용되며, onBufferFull에서 상대방에게 데이터를 계속 보내지 않도록 하고, onBufferDrain에서 데이터 쓰기를 계속합니다.


## 콜백 함수 매개변수

``` $connection ```

연결 개체, 즉 [TcpConnection 인스턴스](../tcp-connection.md)로 클라이언트 연결을 조작하기 위해 사용됨, 예를들면 [데이터 전송](../tcp-connection/send.md), [연결 종료](../tcp-connection/close.md) 등


## 예제

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onBufferFull = function(TcpConnection $connection)
{
    echo "버퍼가 가득 찼으므로 더 이상 전송하지 않습니다.\n";
};
$worker->onBufferDrain = function(TcpConnection $connection)
{
    echo "버퍼가 비었으므로 계속해서 전송합니다.\n";
};
// worker 실행
Worker::runAll();
```

팁: 익명 함수 외에도 [여기](../faq/callback_methods.md)에서 다른 콜백 작성 방법을 확인할 수 있습니다.


## 참고
onBufferFull: 연결의 응용 계층 송신 버퍼가 가득 찼을 때 트리거됨
