# onMessage
## 설명:
```php
callback Connection::$onMessage
```

[Worker::$onMessage](../worker/on-message.md) 콜백과 동일한 기능을 합니다. 차이점은 현재 연결에만 적용되며, 특정 연결에 대한 onMessage 콜백을 설정할 수 있다는 것입니다.

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 클라이언트 연결 이벤트 발생 시
$worker->onConnect = function(TcpConnection $connection)
{
    // 연결의 onMessage 콜백 설정
    $connection->onMessage = function(TcpConnection $connection, $data)
    {
        var_dump($data);
        $connection->send('receive success');
    };
};
// worker 실행
Worker::runAll();
```

위의 코드와 아래의 코드는 동일한 효과를 가집니다.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 모든 연결의 onMessage 콜백 직접 설정
$worker->onMessage = function(TcpConnection $connection, $data)
{
    var_dump($data);
    $connection->send('receive success');
};
// worker 실행
Worker::runAll();
```
