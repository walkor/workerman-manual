# defaultMaxSendBufferSize
## 설명:
```php
static int Connection::$defaultMaxSendBufferSize
```

이 속성은 모든 연결의 기본 응용 계층 전송 버퍼 크기를 설정하는 데 사용되는 전역 정적 속성입니다. 기본값은 `1MB`입니다. `Connection::$defaultMaxSendBufferSize`는 동적으로 설정될 수 있으며, 설정 후에는 이후 생성된 새 연결에만 적용됩니다.

이 속성은 [onBufferFull](../worker/on-buffer-full.md) 콜백에 영향을 미칩니다.

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 모든 연결의 기본 응용 계층 전송 버퍼 크기 설정
TcpConnection::$defaultMaxSendBufferSize = 2*1024*1024;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 현재 연결의 응용 계층 전송 버퍼 크기 설정. 기본값을 덮어씁니다.
    $connection->maxSendBufferSize = 4*1024*1024;
};
// worker 실행
Worker::runAll();
```
