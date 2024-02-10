# maxSendBufferSize
## 설명:
```php
int Connection::$maxSendBufferSize
```

각 연결은 별도의 응용 프로그램 계층 전송 버퍼를 가지고 있으며, 클라이언트의 수신 속도가 서버의 송신 속도보다 느린 경우 데이터는 응용 프로그램 계층 버퍼에 보관되어 전송을 기다립니다.

이 속성은 현재 연결의 응용 프로그램 계층 전송 버퍼 크기를 설정하는 데 사용됩니다. 설정되지 않으면 기본적으로 [Connection::defaultMaxSendBufferSize](default-max-send-buffer-size.md)(1MB)로 설정됩니다.

이 속성은 [onBufferFull](../worker/on-buffer-full.md) 콜백에 영향을 미칩니다.


## 예제

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onConnect = function(TcpConnection $connection)
{
    // 현재 연결의 응용 프로그램 계층 전송 버퍼 크기를 102400바이트로 설정
    $connection->maxSendBufferSize = 102400;
};
// worker 실행
Worker::runAll();
```
