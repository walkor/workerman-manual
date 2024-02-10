# ws/wss 클라이언트로서의 workerman

가끔은 workerman을 ws/wss 프로토콜을 사용하여 특정 서버에 연결하고 상호 작용하기 위해 클라이언트로 작동하도록해야 합니다.
다음은 예제입니다.

## workerman으로 ws 클라이언트로 동작

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){

    $con = new AsyncTcpConnection('ws://echo.websocket.org:80');
    
    // 웹소켓 핸드 셰이크 성공시
    $con->onWebSocketConnect = function(AsyncTcpConnection $con, ) {
        $con->send('hello');
    };

    // 메시지 수신시
    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## workerman으로 wss(ws+ssl) 클라이언트로 동작

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // ssl은 443 포트에 연결해야합니다.
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443');

    // ssl 암호화 방식을 설정하여 wss로 만듭니다.
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```


## workerman으로 wss(ws+ssl) 클라이언트 + 로컬 SSL 인증서

```php
<?php
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/vendor/autoload.php';

$worker = new Worker();

$worker->onWorkerStart = function($worker){
    // 상대 호스트의 로컬 IP 및 포트 및 SSL 인증서를 설정합니다.
    $context_option = array(
        // SSL 옵션, http://php.net/manual/zh/context.ssl.php 참조
        'ssl' => array(
            // 로컬 인증서 경로. PEM 형식이어야하며 로컬 인증서 및 개인 키를 포함해야합니다.
            'local_cert'        => '/your/path/to/pemfile',
            // local_cert 파일의 암호.
            'passphrase'        => 'your_pem_passphrase',
            // 자체 서명 인증서 허용 여부.
            'allow_self_signed' => true,
            // SSL 인증서를 검증해야하는지 여부.
            'verify_peer'       => false
        )
    );

    // ssl은 443 포트에 연결해야합니다.
    $con = new AsyncTcpConnection('ws://echo.websocket.org:443', $context_option);

    // ssl 암호화 방식을 설정하여 wss로 만듭니다.
    $con->transport = 'ssl';

    $con->onWebSocketConnect = function(AsyncTcpConnection $con) {
        $con->send('hello');
    };

    $con->onMessage = function(AsyncTcpConnection $con, $data) {
        echo $data;
    };

    $con->connect();
};

Worker::runAll();
```

## 기타 설정
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Workerman\Protocols\Ws;
use Workerman\Worker;
use Workerman\Connection\AsyncTcpConnection;

$worker = new Worker();
// 프로세스 시작시
$worker->onWorkerStart = function()
{
    // 원격 웹소켓 서버에 websocket 프로토콜로 연결
    $ws_connection = new AsyncTcpConnection("ws://127.0.0.1:1234");
    // 55초마다 opcode가 0x9인 웹소켓 허트비트를 서버로 전송
    $ws_connection->websocketPingInterval = 55;
    // 사용자 정의 HTTP 헤더
    $ws_connection->headers = ['token' => 'value'];
    // 데이터 유형 설정, 기본값은 BINARY_TYPE_BLOB으로 텍스트입니다.
    $ws_connection->websocketType = Ws::BINARY_TYPE_BLOB; // BINARY_TYPE_BLOB는 텍스트이고 BINARY_TYPE_ARRAYBUFFER는 이진입니다.
    // TCP 3-way 핸드쉐이크 완료시
    $ws_connection->onConnect = function($connection){
        echo "tcp connected\n";
    };
    // 웹소켓 핸드쉐이크 완료시
    $ws_connection->onWebSocketConnect = function(AsyncTcpConnection $con, $response) {
        echo $response;
        $con->send('hello');
    };
    // 원격 웹소켓 서버가 메시지를 보낼 때
    $ws_connection->onMessage = function($connection, $data){
        echo "recv: $data\n";
    };
    // 연결 중 오류가 발생할 때, 일반적으로 원격 웹소켓 서버 연결 실패 오류가 발생합니다.
    $ws_connection->onError = function($connection, $code, $msg){
        echo "error: $msg\n";
    };
    // 원격 웹소켓 서버와의 연결이 끊겼을 때
    $ws_connection->onClose = function($connection){
        echo "connection closed and try to reconnect\n";
        // 연결이 끊기면 1초 후 재연결
        $connection->reConnect(1);
    };
    // 위의 각종 콜백을 설정한 후 연결 작업을 실행합니다.
    $ws_connection->connect();
};
Worker::runAll();
```
