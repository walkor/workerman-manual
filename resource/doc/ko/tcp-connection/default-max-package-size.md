# maxPackageSize

## 설명:
```php
static int Connection::$defaultMaxPackageSize
```

이 속성은 전역 정적 속성으로, 각 연결이 수신할 수 있는 최대 데이터 패키지 크기를 설정하는 데 사용됩니다. 설정되지 않으면 기본값은 10MB입니다.

수신된 데이터 패키지를 구문 분석했을 때(프로토콜 클래스의 input 메서드 반환 값이) 패키지 크기가 ```Connection::$defaultMaxPackageSize```보다 크면 비정상 데이터로 간주되어 연결이 종료됩니다.


## 예시


```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

// 각 연결이 최대 1024000 바이트의 데이터 패키지를 수신하도록 설정
TcpConnection::$defaultMaxPackageSize = 1024000;

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->send('hello');
};
// Worker 실행
Worker::runAll();
```
