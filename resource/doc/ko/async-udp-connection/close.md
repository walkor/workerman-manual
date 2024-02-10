```php
void Connection::close(mixed $data = '')
```

안전하게 연결을 종료하고 연결의 "onClose" 콜백을 호출합니다.

UDP는 연결이 없지만, 대응되는 AsyncUdpConnection 객체는 메모리에 계속 유지되므로, 해당 udp 연결 객체를 해제하려면 close 메서드를 호출해야 하며, 그렇지 않으면 해당 udp 연결 객체가 계속해서 메모리에 유지되어 메모리 누수가 발생할 수 있습니다.

## 매개변수

 ``` $data ```

선택적 매개변수이며, 보낼 데이터입니다 (프로토콜을 지정한 경우, 데이터를 자동으로 프로토콜의 encode 메서드로 패킹합니다). 데이터를 전송한 후에 연결을 종료하고 그 후에 "onClose" 콜백을 호출합니다.

데이터 크기는 65507바이트를 초과할 수 없으며, 그렇지 않으면 전송에 실패합니다.

### 예시 

```php
use Workerman\Worker;
use Workerman\Timer;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Connection\UdpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1초 후에 udp 클라이언트를 시작하여 1234 포트에 연결하고 문자열 "hi"를 전송합니다.
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function(AsyncUdpConnection $udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function(AsyncUdpConnection $udp_connection, $data){
            // 서버로부터 반환된 데이터 "hello"를 수신합니다.
            echo "recv $data\r\n";
            // 연결을 종료
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function(UdpConnection $connection, $data)
{
    // AsyncUdpConnection 클라이언트가 보낸 데이터를 수신하여 문자열 "hello"를 반환합니다.
    $connection->send("hello");
};
Worker::runAll();             
```

실행 후에 유사한 내용이 출력됩니다:
```
recv hello
```
