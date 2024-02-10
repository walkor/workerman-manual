# onClose
## 설명:
```php
callback Connection::$onClose
```

이 콜백은 [Worker::$onClose](../worker/on-close.md) 콜백과 동일한 역할을 합니다. 차이점은 현재 연결에만 적용되며, 특정 연결에 대한 onClose 콜백을 설정할 수 있다는 것입니다.

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 연결 이벤트가 발생했을 때 실행
$worker->onConnect = function(TcpConnection $connection)
{
    // 연결의 onClose 콜백 설정
    $connection->onClose = function(TcpConnection $connection)
    {
        echo "connection closed\n";
    };
};
// worker 실행
Worker::runAll();
```

위 코드는 아래와 동일한 효과를 갖습니다.

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 모든 연결의 onClose 콜백 설정
$worker->onClose = function(TcpConnection $connection)
{
    echo "connection closed\n";
};
// worker 실행
Worker::runAll();
```
