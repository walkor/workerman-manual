# connect 메소드

```php
void AsyncUdpConnection::connect()
```
비동기 연결 작업을 실행합니다. 이 메소드는 즉시 반환됩니다.

### 매개변수
매개변수 없음

### 반환값
반환값 없음

### 예시
```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1초 후에 UDP 클라이언트를 시작하여 1234 포트에 연결하고 문자열 "hi"를 전송합니다.
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // 서버에서 반환된 데이터 "hello"를 수신
            echo "recv $data\r\n";
            // 연결 닫기
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnection 클라이언트가 보낸 데이터를 수신하고, 문자열 "hello"를 반환합니다.
    $connection->send("hello");
};
Worker::runAll();             
```

실행 후 다음과 유사한 내용이 출력됩니다:
```text
recv hello
```
