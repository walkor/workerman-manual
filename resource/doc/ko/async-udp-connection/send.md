# send 메서드
```php
void AsyncUdpConnection::send(string $data)
```
비동기 연결 작업을 실행합니다.이 메서드는 즉시 반환됩니다.

### 매개변수
``` $data ```
서버로 전송할 데이터로, 데이터 크기는 65507바이트를 초과할 수 없습니다(udp 단일 데이터 패킷의 최대 전송 크기는 65507바이트입니다). 그렇지 않으면 전송에 실패합니다.

### 반환 값
반환 값이 없습니다.

### 예시

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1초 후에 udp 클라이언트를 시작하고 1234번 포트에 연결하여 문자열 hi를 전송합니다.
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // 서버로부터 반환된 데이터 hello를 수신합니다.
            echo "recv $data\r\n";
            // 연결 종료
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnection 클라이언트가 전송한 데이터를 수신하고, 문자열 hello를 반환합니다.
    $connection->send("hello");
};
Worker::runAll();             
```

실행 후 출력:
```php
recv hello
```
