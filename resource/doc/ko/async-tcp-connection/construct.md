# __construct 메서드

```php
void AsyncTcpConnection::__construct(string $remote_address, $context_option = null)
```

비동기 연결 객체를 생성합니다.

AsyncTcpConnection을 사용하면 Workerman이 클라이언트로 원격 서버에 비동기적으로 연결하고, send 인터페이스와 onMessage 콜백을 통해 연결된 데이터를 비동기적으로 보내고 처리할 수 있습니다.

## 매개변수
매개변수:``` remote_address ```

연결 주소, 예를 들어
```
tcp://www.baidu.com:80
ssl://www.baidu.com:443
ws://echo.websocket.org:80
frame://192.168.1.1:8080
text://192.168.1.1:8080
```

매개변수:``` $context_option ```

``` (workerman >= 3.3.5 이상 지원) ```

소켓 컨텍스트를 설정하기 위해 사용됩니다. ```bindto```를 사용하여 외부 네트워크에 어떤(IP 주소 카드) IP 및 포트로 연결할지, SSL 인증서 설정 등을 지정합니다.

참고: [stream_context_create](https://php.net/manual/en/function.stream-context-create.php), [소켓 컨텍스트 옵션](https://php.net/manual/zh/context.socket.php), [SSL 컨텍스트 옵션](https://php.net/manual/zh/context.ssl.php)

## 주의사항

현재 AsyncTcpConnection이 지원하는 프로토콜은 [tcp](https://baike.baidu.com/subview/32754/8048820.htm), [ssl](https://baike.baidu.com/view/525499.htm), [ws](appendices/about-ws.md), [frame](appendices/about-frame.md), [text](appendices/about-text.md)입니다.

또한 사용자 정의 프로토콜을 지원하며, [사용자 정의 프로토콜 구현 방법](../protocols/how-protocols.md)을 참조하십시오.

여기서 [ssl](https://baike.baidu.com/view/525499.htm)은 Workerman>=3.3.4 가 필요하며 [openssl 확장 기능](https://php.net/manual/zh/book.openssl.php)이 필요합니다.

현재 AsyncTcpConnection은 [http](https://baike.baidu.com/view/9472.htm) 프로토콜을 지원하지 않습니다.

`new AsyncTcpConnection('ws://...')`을 사용하여 브라우저로 원격 웹 소켓 서버에 연결한 것처럼 Workerman에서 원격 웹소켓 서버에 연결할 수 있지만, `new AsyncTcpConnection('websocket://...')` 형식으로 웹소켓 연결을 시작할 수는 없습니다.

## 예시

### 예시 1: 외부 http 서비스에 대한 비동기 액세스
```php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$task = new Worker();
// 프로세스 시작시 www.baidu.com으로 비동기 연결을 설정하고 데이터를 보내고 받습니다.
$task->onWorkerStart = function($task)
{
    // 직접적인 http 지정은 지원하지 않지만, tcp를 사용하여 http 프로토콜 데이터를 보낼 수 있습니다
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // 연결 성공 시, http 요청 데이터를 보냅니다.
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

// Worker 실행
Worker::runAll();
```

### 예시 2: 외부 웹소켓 서비스에 대한 비동기 액세스 및 로컬 IP 및 포트 설정
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 상대방 호스트에 연결할 로컬 IP 및 포트 설정 (각 소켓 연결은 로컬 포트 하나를 사용합니다)
    $context_option = array(
        'socket' => array(
            // IP는 로컬 네트워크 카드 IP 여야 하며 상대방 호스트에 액세스할 수 있어야 하므로 그렇지 않으면 무효화됩니다.
            'bindto' => '114.215.84.87:2333',
        ),
    );

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80', $context_option);

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('안녕하세요');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

### 예시 3: 외부 wss 포트에 대한 비동기 액세스 및 로컬 SSL 인증서 설정
```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
use Workerman\Connection\TcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 상대방 호스트에 액세스할 로컬 IP 및 포트 및 SSL 인증서 설정
    $context_option = array(
        'socket' => array(
            // IP는 로컬 네트워크 카드 IP 여야 하며 상대방 호스트에 액세스할 수 있어야 하므로 그렇지 않으면 무효화됩니다.
            'bindto' => '114.215.84.87:2333',
        ),
        // SSL 옵션을 참조하세요. https://php.net/manual/zh/context.ssl.php
        'ssl' => array(
            // 로컬 인증서 경로. PEM 형식이어야 하고 로컬 인증서 및 비밀키를 포함해야 합니다.
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert 파일의 암호.
            'passphrase'        => 'your_pem_passphrase',
            // 자체 서명된 인증서를 허용할 것인지 여부.
            'allow_self_signed' => true,
            // SSL 인증서를 검증해야 하는지 여부.
            'verify_peer'       => false
        )
    );

    // 비동기 연결 설정
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // SSL 암호화 방식으로 설정
    $con->transport = 'ssl';

    $con->onConnect = function(AsyncTcpConnection $con) {
        $con->send('안녕하세요');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```
