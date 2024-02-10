# close
## 설명:
```php
void Connection::close(mixed $data = '')
```

안전하게 연결을 종료합니다.

close를 호출하면 연결이 종료되기 전에 보낼 데이터가 모두 전송될 때까지 기다리고, 연결의 `onClose` 콜백을 트리거합니다.

## 매개변수

``` $data ```

선택 사항입니다. 전송할 데이터 (프로토콜이 지정된 경우 자동으로 프로토콜의 encode 메서드를 사용하여 ```$data``` 데이터를 패킹)를 지정합니다. 데이터가 전송된 후 연결이 종료되며, 그 후에는 onClose 콜백이 트리거됩니다.

## 예시

```php
use Workerman\Worker;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('websocket://0.0.0.0:8484');
$worker->onMessage = function(TcpConnection $connection, $data)
{
    $connection->close("hello\n");
};
// worker 실행
Worker::runAll();
```
