# transport 속성
```요구 (workerman >= 3.3.4)```

전송 속성을 설정하며, 옵션 값은 [tcp](https://baike.baidu.com/subview/32754/8048820.htm)와 [ssl](https://baike.baidu.com/view/525499.htm)로 기본값은 tcp입니다.

transport를 [ssl](https://baike.baidu.com/view/525499.htm)로 설정할 경우 PHP에 [openssl extension](https://php.net/manual/zh/book.openssl.php)이 설치되어 있어야 합니다.

Workerman을 클라이언트로 사용하여 서버에 ssl 암호화 연결(https 연결, wss 연결 등)을 시작할 경우 이 옵션을 'ssl'로 설정해야 합니다. 아래 예시 참고.

### 예시 (https 연결)
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 워커가 시작될 때 www.baidu.com으로의 비동기 연결 객체를 생성하고 데이터를 보내 데이터를 받습니다.
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:443');

    // ssl 암호화 연결로 설정합니다.
    $connection_to_baidu->transport = 'ssl';

    $connection_to_baidu->onConnect = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "연결 성공\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function(AsyncTcpConnection $connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function(AsyncTcpConnection $connection_to_baidu)
    {
        echo "연결 종료\n";
    };
    $connection_to_baidu->onError = function(AsyncTcpConnection $connection_to_baidu, $code, $msg)
    {
        echo "에러 코드:$code 메시지:$msg\n";
    };
    $connection_to_baidu->connect();
};

// 워커 실행
Worker::runAll();
```
