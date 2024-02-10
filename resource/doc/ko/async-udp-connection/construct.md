# __construct 메서드

```php
void AsyncUdpConnection::__construct(string $remote_address)
```
UDP 연결 객체를 생성합니다.

AsyncUdpConnection은 Workerman을 클라이언트로 사용하여 원격 서버와 UDP 데이터를 전송할 수 있게 해줍니다.

## 매개변수
매개변수: ``` remote_address ```

연결할 주소, 예를 들면 
``` udp://192.168.1.1:1234 ```
``` frame://192.168.1.1:8080 ```
``` text://192.168.1.1:8080 ```

## 예제

```php
use Workerman\Worker;
use Workerman\Connection\AsyncUdpConnection;
use Workerman\Timer;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker('udp://0.0.0.0:1234');
$worker->onWorkerStart = function(){
    // 1초 후에 UDP 클라이언트를 시작하여 1234 포트에 연결하고 문자열 'hi'를 보냅니다.
    Timer::add(1, function(){
        $udp_connection = new AsyncUdpConnection('udp://127.0.0.1:1234');
        $udp_connection->onConnect = function($udp_connection){
            $udp_connection->send('hi');
        };
        $udp_connection->onMessage = function($udp_connection, $data){
            // 서버에서 반환된 데이터 'hello'를 받습니다.
            echo "recv $data\r\n";
            // 연결 종료
            $udp_connection->close();
        };
        $udp_connection->connect();
    }, null, false);
};
$worker->onMessage = function($connection, $data)
{
    // AsyncUdpConnection 클라이언트가 전송한 데이터를 받아서 'hello' 문자열을 반환합니다.
    $connection->send("hello");
};
Worker::runAll();             
```

실행 후 다음과 같이 출력됩니다.
``` 
recv hello
```
